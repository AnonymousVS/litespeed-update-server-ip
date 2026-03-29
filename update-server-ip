#!/bin/bash
# WP-Toolkit-CleanUP by AnonymousVS
# https://github.com/AnonymousVS/WP-Toolkit-CleanUP

FLAG=".wp-cleanup-done"
LOG="/var/log/wp-cleanup.log"
PARALLEL_JOBS=8

chmod +x "$0"

if ! crontab -l 2>/dev/null | grep -q "wp-cleanup-auto.sh"; then
  (crontab -l 2>/dev/null; echo "0 1 * * * /usr/bin/flock -n /tmp/wp-cleanup.lock /usr/local/sbin/wp-cleanup-auto.sh") | crontab -
  echo "Cron added: runs daily at 01:00"
fi

if [ ! -f /etc/logrotate.d/wp-cleanup ]; then
  cat > /etc/logrotate.d/wp-cleanup << 'EOF'
/var/log/wp-cleanup.log {
    daily
    rotate 7
    compress
    missingok
    notifempty
}
EOF
  echo "Logrotate configured"
fi

spinner() {
  local pid=$1
  local delay=0.1
  local frames=('⠋' '⠙' '⠹' '⠸' '⠼' '⠴' '⠦' '⠧' '⠇' '⠏')
  local i=0
  while kill -0 "$pid" 2>/dev/null; do
    local count=0
    [ -f "$LOG" ] && count=$(grep -c "Cleaned:" "$LOG" 2>/dev/null || echo 0)
    printf "\r${frames[$i]} Processing... cleaned: %s sites" "$count"
    i=$(( (i+1) % ${#frames[@]} ))
    sleep $delay
  done
  local total=0
  [ -f "$LOG" ] && total=$(grep -c "Cleaned:" "$LOG" 2>/dev/null || echo 0)
  printf "\r✅ Done! Total cleaned: %s sites\n" "$total"
}

echo "[$(date '+%Y-%m-%d %H:%M:%S')] === START ===" >> "$LOG"

cleanup_site() {
  local WP_PATH="$1"
  local FLAG="$2"
  local LOG="$3"

  wp plugin delete \
    akismet hello \
    All-In-One-WP-Migration-With-Import-master \
    imunify-security \
    malcare-security \
    sucuri-scanner \
    --path="$WP_PATH" --allow-root --quiet \
    --skip-plugins --skip-themes >/dev/null 2>&1

  wp theme delete \
    twentytwentythree twentytwentyfour twentytwentytwo twentytwentyone twentytwenty \
    --path="$WP_PATH" --allow-root --quiet \
    --skip-plugins --skip-themes >/dev/null 2>&1

  wp config set CORE_UPGRADE_SKIP_NEW_BUNDLED true \
    --raw --path="$WP_PATH" --allow-root --quiet \
    --skip-plugins --skip-themes >/dev/null 2>&1

  touch "$WP_PATH/$FLAG"
  echo "[$(date '+%Y-%m-%d %H:%M:%S')] Cleaned: $WP_PATH" >> "$LOG"
}

export -f cleanup_site

(
  MAIN_DOMAINS=$(awk -F': ' '{print $1}' /etc/trueuserdomains | sed 's/ //g')

  grep -v "\.cp:" /etc/userdomains \
    | grep -v "nobody" \
    | grep -v "\*" \
    | while IFS=': ' read -r domain user; do
        domain=$(echo "$domain" | tr -d ' ')
        user=$(echo "$user" | tr -d ' ')

        # ข้าม main domain
        echo "$MAIN_DOMAINS" | grep -qx "$domain" && continue

        # ข้าม cPanel internal subdomain
        echo "$MAIN_DOMAINS" | while read -r md; do
          echo "$domain" | grep -q "\.${md}$" && echo "SKIP" && break
        done | grep -q "SKIP" && continue

        # หา wp-config.php ทั้ง 2 path
        for wp_config in \
          "/home/$user/$domain/wp-config.php" \
          "/home/$user/public_html/$domain/wp-config.php"; do
          if [ -f "$wp_config" ]; then
            WP_PATH=$(dirname "$wp_config")
            [ -f "$WP_PATH/$FLAG" ] && continue
            echo "$WP_PATH"
          fi
        done

      done \
    | sort -u \
    | xargs -P "$PARALLEL_JOBS" -I{} bash -c \
      'cleanup_site "$@"' _ {} "$FLAG" "$LOG"
) &

CLEANUP_PID=$!
spinner $CLEANUP_PID
wait $CLEANUP_PID

echo "[$(date '+%Y-%m-%d %H:%M:%S')] === DONE ===" >> "$LOG"
