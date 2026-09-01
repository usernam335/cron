# CRON
Описание домашнего задания:

Написать скрипт для CRON, который раз в час формирует отчёт и отправляет его на заданную почту.

# 1. Скрипт для анализа логов 

    #!/bin/bash


    LOG_FILE="/var/log/nginx/access.log"
    REPORT_FILE="/tmp/nginx_report.txt"
    TOP_COUNT=10

    if [ ! -f "$LOG_FILE" ]; then
        echo "ОШИБКА: Файл $LOG_FILE не найден!"
        exit 1
    fi

    echo "Анализ лога: $LOG_FILE"
    echo "Размер: $(du -h "$LOG_FILE" | cut -f1)"


    TEMP_DIR=$(mktemp -d)
    trap "rm -rf $TEMP_DIR" EXIT

    safe_sort() {
        sort -S 1G -T "$TEMP_DIR" "$1" 2>/dev/null || sort "$1"
    }

    echo " 1. IP-АДРЕСА"

    awk '{print $1}' "$LOG_FILE" > "$TEMP_DIR/ips.txt"


    safe_sort "$TEMP_DIR/ips.txt" | uniq -c | sort -rn | head -"$TOP_COUNT"

    echo  "2. URL"

    awk '{print $7}' "$LOG_FILE" > "$TEMP_DIR/urls.txt"
    safe_sort "$TEMP_DIR/urls.txt" | uniq -c | sort -rn | head -"$TOP_COUNT"

    echo  "3. HTTP КОДЫ"

    awk '{print $9}' "$LOG_FILE" > "$TEMP_DIR/codes.txt"
    safe_sort "$TEMP_DIR/codes.txt" | uniq -c | sort -rn

    echo "4. ОШИБКИ"

    awk '$9 ~ /^[45]/ {print $9}' "$LOG_FILE" > "$TEMP_DIR/errors.txt"
    if [ ! -s "$TEMP_DIR/errors.txt" ]; then
        echo "Ошибок не найдено!"
    else
        safe_sort "$TEMP_DIR/errors.txt" | uniq -c | sort -rn
    fi

    echo "5. 404 ОШИБКИ"

    awk '$9 == "404" {print $7}' "$LOG_FILE" > "$TEMP_DIR/notfound.txt"
    safe_sort "$TEMP_DIR/notfound.txt" | uniq -c | sort -rn | head -"$TOP_COUNT"


    {
        echo "ОТЧЕТ АНАЛИЗА ЛОГОВ"
        echo "Дата: $(date)"
        echo "----------------------------------------"

        echo "ТОП IP:"
        safe_sort "$TEMP_DIR/ips.txt" | uniq -c | sort -rn | head -"$TOP_COUNT"

        echo ""
        echo "ТОП URL:"
        safe_sort "$TEMP_DIR/urls.txt" | uniq -c | sort -rn | head -"$TOP_COUNT"

        echo ""
        echo "HTTP КОДЫ:"
        safe_sort "$TEMP_DIR/codes.txt" | uniq -c | sort -rn

        echo ""
        echo "ОШИБКИ:"
        if [ -s "$TEMP_DIR/errors.txt" ]; then
            safe_sort "$TEMP_DIR/errors.txt" | uniq -c | sort -rn
        else
            echo "Нет ошибок"
        fi
    } > "$REPORT_FILE"

    echo ""
    echo "Готово! Отчет: $REPORT_FILE"

# 2. НАстройка CRON для отправки отчета на почту каждый час:

Для настройки cron нужно ввести команду:

    crontab -e

Добавим эти строки в crontab для отправки отчета :



<img width="977" height="694" alt="Screenshot_2" src="https://github.com/user-attachments/assets/e2fc7810-2d76-49d6-a89a-4c220d97d9fe" />



    MAILTO="username@gmail.com"

    0 * * * * * /root/otus-bash.sh

    0 * * * * * cat /tmp/nginx_report.txt | mail -s "TEST" $(date) username@gmail.com    


# 3. Результат

<img width="648" height="534" alt="Screenshot_3" src="https://github.com/user-attachments/assets/fcfe1c20-6fd3-495e-a81b-93a380bc5977" />





