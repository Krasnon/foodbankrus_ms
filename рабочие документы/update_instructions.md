# Инструкция по автоматическому обновлению документации МойСклад

## Описание

Данная инструкция описывает процесс автоматического обновления документации МойСклад через извлечение данных с официального сайта help.moysklad.ru.

## Возможности автоматизации

### 1. Извлечение текстовой информации
- Получение HTML-страниц через `curl`
- Парсинг и извлечение текста
- Структурирование в markdown формат
- Обновление соответствующих разделов документации

### 2. Работа с изображениями
- Скачивание изображений с веб-страниц
- Обработка и оптимизация изображений
- Организация в структурированные папки
- Обновление ссылок в документации

### 3. Структурирование данных
- Автоматическое создание оглавлений
- Генерация индексов и ссылок
- Обновление README файлов
- Создание навигации между разделами

## Команды для работы

### Базовые команды curl

```bash
# Получение HTML-страницы
curl -s "https://help.moysklad.ru/hc/ru/articles/360000031943-Как-создать-приемку" > page.html

# Извлечение текста (требует установки html2text)
curl -s "URL" | html2text > extracted_text.md

# Скачивание изображения
curl -O "https://help.moysklad.ru/images/screenshot.png"
```

### Обработка изображений

```bash
# Конвертация форматов
convert screenshot.png screenshot.jpg

# Изменение размера
convert screenshot.png -resize 800x600 screenshot_small.png

# Создание превью
convert screenshot.png -resize 200x150 thumbnail.png

# Оптимизация для веба
convert screenshot.png -quality 85 screenshot_optimized.jpg
```

### Структурирование файлов

```bash
# Создание структуры папок
mkdir -p manual/images/{interface,documents,procedures,reports,diagrams}

# Перемещение файлов
mv screenshot.png manual/images/procedures/

# Обновление ссылок в markdown
sed -i 's|old_path|new_path|g' *.md
```

## Процесс обновления документации

### Шаг 1: Определение URL для обновления
```bash
# Примеры URL для различных разделов
RECEIPT_URL="https://help.moysklad.ru/hc/ru/articles/360000031943-Как-создать-приемку"
SHIPMENT_URL="https://help.moysklad.ru/hc/ru/articles/360000031944-Как-создать-отгрузку"
CONTRAGENT_URL="https://help.moysklad.ru/hc/ru/articles/360000031945-Как-создать-контрагента"
```

### Шаг 2: Извлечение контента
```bash
# Получение HTML
curl -s "$RECEIPT_URL" > temp_page.html

# Извлечение текста
html2text temp_page.html > extracted_content.md

# Очистка временных файлов
rm temp_page.html
```

### Шаг 3: Обработка изображений
```bash
# Поиск изображений в HTML
grep -o 'https://[^"]*\.png\|https://[^"]*\.jpg' temp_page.html > image_urls.txt

# Скачивание изображений
while read url; do
    filename=$(basename "$url")
    curl -O "$url"
    mv "$filename" "manual/images/procedures/"
done < image_urls.txt
```

### Шаг 4: Обновление документации
```bash
# Обновление соответствующего файла
# Например, для приемки - 13_product_receipt.md
cp extracted_content.md manual/13_product_receipt.md

# Обновление ссылок на изображения
sed -i 's|help.moysklad.ru/images/|../images/procedures/|g' manual/13_product_receipt.md
```

## Структура файлов для обновления

### Соответствие URL и файлов документации

| URL раздел | Файл документации | Описание |
|------------|-------------------|----------|
| Приемка | `manual/13_product_receipt.md` | Создание приемок |
| Отгрузка | `manual/15_distribution_fixation_moysklad.md` | Создание отгрузок |
| Контрагенты | `manual/01_new_donor_format.md` | Создание контрагентов |
| Товары | `manual/09_stock_management.md` | Управление товарами |
| Нумерация | `manual/05_contract_numbering.md` | Нумерация документов |
| Печать | `manual/19_document_control_moysklad.md` | Печать документов |
| Уведомления | `manual/20_communication_block.md` | Фильтры и уведомления |

### Папки для изображений

| Тип изображения | Папка | Описание |
|-----------------|-------|----------|
| Интерфейс | `images/interface/` | Скриншоты интерфейса МойСклад |
| Документы | `images/documents/` | Примеры документов и форм |
| Процедуры | `images/procedures/` | Пошаговые инструкции |
| Отчеты | `images/reports/` | Примеры отчетов |
| Диаграммы | `images/diagrams/` | Схемы и диаграммы |

## Автоматизация процесса

### Скрипт для полного обновления

```bash
#!/bin/bash
# update_moysklad_docs.sh

# Список URL для обновления
declare -A URLS=(
    ["receipt"]="https://help.moysklad.ru/hc/ru/articles/360000031943-Как-создать-приемку"
    ["shipment"]="https://help.moysklad.ru/hc/ru/articles/360000031944-Как-создать-отгрузку"
    ["contragent"]="https://help.moysklad.ru/hc/ru/articles/360000031945-Как-создать-контрагента"
)

# Функция обновления раздела
update_section() {
    local section=$1
    local url=$2
    local file="manual/${section}.md"
    
    echo "Обновление раздела: $section"
    
    # Получение контента
    curl -s "$url" > temp_page.html
    
    # Извлечение текста
    html2text temp_page.html > temp_content.md
    
    # Обработка изображений
    grep -o 'https://[^"]*\.png\|https://[^"]*\.jpg' temp_page.html | while read img_url; do
        filename=$(basename "$img_url")
        curl -s -O "$img_url"
        mv "$filename" "manual/images/procedures/"
    done
    
    # Обновление файла
    cp temp_content.md "$file"
    
    # Очистка
    rm temp_page.html temp_content.md
}

# Обновление всех разделов
for section in "${!URLS[@]}"; do
    update_section "$section" "${URLS[$section]}"
done
```

## Контроль качества

### Проверка обновлений
```bash
# Проверка синтаксиса markdown
markdownlint manual/*.md

# Проверка ссылок на изображения
find manual -name "*.md" -exec grep -l "images/" {} \;

# Проверка структуры файлов
tree manual/
```

### Резервное копирование
```bash
# Создание бэкапа перед обновлением
cp -r manual/ manual_backup_$(date +%Y%m%d_%H%M%S)/

# Восстановление при необходимости
# cp -r manual_backup_YYYYMMDD_HHMMSS/* manual/
```

## Рекомендации

### Для регулярных обновлений:
1. **Создайте расписание** обновлений (например, раз в месяц)
2. **Ведите журнал** изменений
3. **Проверяйте качество** обновленной документации
4. **Тестируйте** инструкции на реальных пользователях

### Для критических обновлений:
1. **Создавайте бэкапы** перед обновлением
2. **Тестируйте** на копии документации
3. **Проводите ревью** изменений
4. **Уведомляйте** пользователей об изменениях

## Контакты для поддержки

- **Техническая поддержка**: it@foodbank.ru
- **Документация МойСклад**: help.moysklad.ru
- **API МойСклад**: api.moysklad.ru

---

*Инструкция обновляется по мере развития возможностей автоматизации* 