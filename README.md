# CV Practice — Классификация мусора для сортировки отходов (TrashNet)

Сравнение **5 разных архитектур**
классификаторов, замер качества/скорости/размера, лучшая модель в демо-приложении
с рекомендацией по сортировке, отчёт PDF/Excel. Железо: **Apple Silicon (MPS)**.

## Прикладная задача
Определение типа отхода по фотографии (6 классов: **cardboard, glass, metal,
paper, plastic, trash**) для автоматической сортировки и направления на
переработку. Датасет — TrashNet (Stanford). Польза: ускорение сортировки,
снижение доли несортированных отходов.

## Структура
```
cv_practice/
├── configs/            # data.yaml + (создаётся) norm_stats.json, конфиги моделей (ШАГ 2)
├── data/
│   ├── raw/            # СЮДА распаковать датасет (в .gitignore)
│   └── splits/         # manifest.csv, class_map.json, статистика, график
├── notebooks/          # черновой анализ
├── src/
│   ├── utils/          # seed, device(MPS), config
│   ├── data/           # prepare_data.py, dataset.py
│   ├── train.py        # ШАГ 2
│   ├── evaluate.py     # ШАГ 3
│   └── inference.py    # ШАГ 4
├── models/             # сохранённые веса (в .gitignore)
├── runs/               # логи/метрики запусков
├── demo/               # ШАГ 4 — Gradio-приложение
├── report/             # ШАГ 5 — PDF/Excel + примеры
├── requirements.txt
└── setup.sh
```

## Дорожная карта (= требования задания)
| Шаг | Файл | Что закрывает |
|----|------|---------------|
| 1 ✅ | `src/data/prepare_data.py`, `dataset.py` | split (seed=42), нормализация, анти-утечка, структура |
| 2 | `configs/*.yaml`, `src/train.py` | обучение 5 архитектур (ResNet, DenseNet, MobileNetV3, EfficientNet, ViT) |
| 3 | `src/evaluate.py` | accuracy/precision/recall/F1, confusion matrix, FPS, размер модели, таблица |
| 4 | `demo/app.py` | UI: фото + предсказание + уверенность + статистика |
| 5 | `src/report.py` | история в JSON/SQLite + PDF/Excel + 3 удачных/3 ошибочных примера |

## Установка
```bash
bash setup.sh                 # создаст .venv и поставит зависимости
source .venv/bin/activate
```

## Скачать датасет (TrashNet, ~42 МБ, без авторизации)
Нужен формат ImageFolder: `<raw_dir>/<класс>/*.jpg`. Источник — Hugging Face
(зеркало TrashNet, лицензия MIT):
```bash
URL="https://huggingface.co/datasets/garythung/trashnet/resolve/main/dataset-resized.zip"
curl -L --http1.1 -o data/raw/garbage.zip "$URL"
unzip -q data/raw/garbage.zip -d data/raw/garbage && rm data/raw/garbage.zip
# классы окажутся в data/raw/garbage/dataset-resized/{cardboard,glass,metal,paper,plastic,trash}
```
Путь к данным задан в `configs/data.yaml` → `raw_dir: data/raw/garbage/dataset-resized`.

## ШАГ 1 — подготовка данных
```bash
python -m src.data.prepare_data --config configs/data.yaml
# быстрый смоук-тест на маленькой подвыборке:
python -m src.data.prepare_data --max-per-class 200
```
На выходе: `data/splits/manifest.csv` — **единый split для всех 5 моделей**,
а также `class_map.json`, `dataset_stats.json`, `class_distribution.png`,
`configs/norm_stats.json`.
