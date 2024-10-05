#!/bin/bash
clear

# Function สำหรับส่งข้อความผ่าน Line Notify
send_line_notification() {
  message=$1
  token=""  # แทนที่ด้วย Token ของ Line Notify ของคุณ
  curl -X POST -H "Authorization: Bearer $token" -F "message=$message" https://notify-api.line.me/api/notify > /dev/null 2>&1
}

# Function สำหรับตรวจสอบจำนวนออนไลน์และส่งข้อมูลไปยังไฟล์
fun_online() {
  limite=$(cat /etc/openvpn/limite)  # แทนที่ด้วยค่าที่ต้องการเซ็ตให้เป็นค่าของ Limite
  _ons=$(ps -x | grep sshd | grep -v root | grep -v system2 | grep priv | wc -l)
  [[ -e /etc/openvpn/openvpn-status.log ]] && _onop=$(grep -c "10.8" /etc/openvpn/openvpn-status.log) || _onop="0"
  [[ -e /etc/default/dropbear ]] && _drp=$(ps aux | grep dropbear | grep -v grep | wc -l) _ondrp=$(($_drp - 1)) || _ondrp="0"
  _onli=$(($_ons + $_onop + $_ondrp))
  _onlin=$(printf '%-1s' "$_onli")
  CURRENT_ONLINES="$(echo -e "${_onlin}" | sed -e 's/[[:space:]]*$//')"
  echo "[ {\"onlines\":\"$CURRENT_ONLINES\",\"limite\":\"$limite\"} ]" > /var/www/html/server/online_app.json
  echo "[ {\"onlines\":\"$CURRENT_ONLINES\",\"limite\":\"$limite\"} ]" > /home/public_html/server/online_app.json
  echo "[ {\"onlines\":\"$CURRENT_ONLINES\",\"limite\":\"$limite\"} ]" > /home/public_html/online_app.json
  # ตรวจสอบเวลาที่แจ้งเตือนล่าสุด
  last_notification_time_file="/var/www/html/server/last_notification_time.txt"
  last_notification_time=0
  if [ -e "$last_notification_time_file" ]; then
    last_notification_time=$(cat "$last_notification_time_file")
  fi

  # ตรวจสอบว่าผ่านไป 1 ชั่วโมงหรือไม่ (3600 วินาที)
  current_time=$(date +%s)
  time_diff=$((current_time - last_notification_time))
  hour_in_seconds=1800

  if [ "$time_diff" -ge "$hour_in_seconds" ]; then
    # ส่งข้อความ Line Notify
    send_line_notification "จำนวนออนไลน์: $CURRENT_ONLINES / Limite: $limite"

    # เก็บเวลาปัจจุบันเป็นเวลาแจ้งเตือนล่าสุด
    echo "$current_time" > "$last_notification_time_file"
  fi
}

# คำสั่งในส่วนของ while true
while true; do
  echo 'verificando...'
  fun_online > /dev/null 2>&1
  sleep 15s
done
