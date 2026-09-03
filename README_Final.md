# AutoFlow AI — Autonomous Data-to-Decision Platform

> **От данных к решению — без ручного прохождения ML pipeline.**

[![Status](https://img.shields.io/badge/status-in--development-orange)]()
[![Python](https://img.shields.io/badge/Python-3.x-blue)]()
[![DuckDB](https://img.shields.io/badge/DuckDB-Big%20Data-yellow)]()
[![ML](https://img.shields.io/badge/ML-production--oriented-green)]()
[![Kaggle](https://img.shields.io/badge/Runtime-Kaggle-20BEFF)]()

## Автор проекта

**Юлия Прохорова** — [Junior ML-Engineer](https://drive.google.com/file/d/11ycGzexiiByO1bbBUde2KAQzRud-tr8A/view?usp=sharing) 
* Email: prohorova_j@inbox.ru  
* Telegram: @j_u_l_i_p_r_o_k_h_o_r_o_v_a

---

## Проект

**AutoFlow AI** — production-oriented платформа для автоматизированной обработки больших потоков данных, интеллектуальной классификации и принятия эксплуатационных решений на основе состояния данных и модели.

Проект построен не как отдельный ML-эксперимент, а как сквозной **autonomous data-to-decision pipeline**, в котором система самостоятельно проходит путь:

```text
RAW DATA
   ↓
Data Audit
   ↓
Quality Control
   ↓
Forensic Deduplication
   ↓
Canonical Dataset
   ↓
Temporal Train / Validation / Test
   ↓
Feature Pipeline
   ↓
Baseline Model
   ↓
Error Diagnostics
   ↓
Model Optimization
   ↓
Accepted Production Model
   ↓
Inference
   ↓
Confidence
   ↓
Automated Routing
   ↓
Drift Monitoring
   ↓
Automated Decision
   ↓
Persistent State & Recovery
   ↓
Production Interface
   ↓
Final Model Audit
```

Основной сценарий проекта — автоматическая классификация обращений граждан на основе открытого датасета **NYC 311 Service Requests**.

---

# Для кого и зачем нужен AutoFlow AI

AutoFlow AI создавался не только для Data Scientist или ML Engineer.

Его практическая ценность особенно хорошо видна в работе **обычного аналитика или офисного сотрудника**, который регулярно получает новые Excel/CSV/Parquet-выгрузки и вынужден выполнять одну и ту же последовательность действий:

```text
получить файл
   ↓
проверить колонки
   ↓
проверить пропуски и ошибки
   ↓
разобраться с дубликатами
   ↓
запустить обработку
   ↓
получить прогнозы
   ↓
отделить понятные случаи от сомнительных
   ↓
вручную проверить часть записей
   ↓
понять, не изменились ли данные
   ↓
решить, можно ли продолжать работу
```

На практике такая рутина занимает время и создаёт риск человеческой ошибки.

**AutoFlow AI переносит эту повторяющуюся техническую работу в автоматический контур.**

Пользователь работает с бизнес-отчётом, а система сама выполняет внутреннюю техническую цепочку:

```text
Новый отчёт
    ↓
Проверка файла и структуры
    ↓
Применение принятой модели
    ↓
Оценка уверенности
    ↓
Автоматическая маршрутизация
    ↓
Контроль изменений данных
    ↓
Эксплуатационное решение
```

### Что в итоге получает обычный пользователь

Не нужно вручную управлять отдельными ML-этапами.

Не нужно каждый раз решать, какую модель загрузить.

Не нужно проверять все строки вручную.

Не нужно самостоятельно сравнивать текущие данные с предыдущим состоянием.

Система сама отделяет случаи, которые можно обработать автоматически, от случаев, где требуется человек:

```text
AUTO_ROUTE
    ↓
можно обработать автоматически

REVIEW
    ↓
требуется дополнительная проверка

HUMAN_REVIEW
    ↓
решение передаётся человеку
```

Таким образом, **человек остаётся ответственным за важные решения, но перестаёт быть оператором каждой отдельной технической операции**.

---

# Бизнес-задача

В традиционном ML workflow после обучения модели значительная часть работы остаётся ручной:

- подготовить новый файл;
- проверить его структуру;
- убедиться, что используется правильная версия модели;
- запустить inference;
- проверить confidence;
- определить, какие записи можно обработать автоматически;
- отправить сомнительные случаи человеку;
- проверить drift;
- решить, требуется ли переобучение;
- сохранить результаты запуска.

**AutoFlow AI автоматизирует именно этот эксплуатационный контур.**

Цель проекта — сделать так, чтобы модель была не конечной точкой ноутбука, а частью работающей системы.

---

# Final result

Финальный запуск выполнен на snapshot датасета NYC 311 Service Requests.

### Исходные данные

| Parameter | Value |
|---|---:|
| Records | 21,960,000 |
| Source columns | 46 |
| CSV size | 13.68 GiB |
| Target | `Complaint Type` |
| Canonical observations | 209,999 |
| Canonical target classes | 147 |

Важно: исходные **21,960,000 строк не следует интерпретировать как 21.96 млн независимых обращений**.

Forensic-анализ показал, что в исходном массиве содержится **21,750,001 повторная строка**, поэтому дальнейшая ML-часть выполняется на Canonical Dataset из **209,999 уникальных наблюдений**.

### Canonical split

```text
TRAIN       146,999
VALIDATION   31,500
TEST         31,500
-------------------
TOTAL       209,999
```

Разбиение выполнено без случайного перемешивания, с контролем временной целостности.

---

# Model result

Canonical baseline:

- Accuracy: **0.969489**
- Validation Macro-F1: **0.750141**
- Validation Macro-Recall: **0.765271**
- Validation Weighted-F1: **0.968760**

Принятая модель:

`SmoothedWeights_4Epochs_Reg3e5`

### Improvement

- Validation Macro-F1: **0.867885**
- Validation Macro-Recall: **0.866548**
- Validation Weighted-F1: **0.980742**
- Improvement over baseline: **+0.117744**

Таким образом, модель improvement увеличил Macro-F1 с **0.750141 до 0.867885**.

Важно, что модель выбиралась **только по Validation Macro-F1**.

`TEST` не использовался для выбора модели или настройки параметров.

### Независимая Blind Test

После завершения model selection выбранная модель была отдельно проверена на TEST:

- Accuracy: **0.981702**
- Macro-F1: **0.818541**
- Macro-Recall: **0.811188**
- Weighted-F1: **0.980268**

Это независимая контрольная оценка уже выбранной модели.

---

# Финальный Before → After audit

Step 20 сравнивает baseline и accepted improved model на **одном и том же Validation Dataset**.

Для обеих confusion matrix используются:

- одинаковый набор Top-20 классов;
- одинаковый порядок классов;
- одинаковые данные;
- одинаковый feature contract.

Меняется только модель.

### Результат

```text
Baseline errors
961
   ↓
Improved model
560
```

Количество ошибок уменьшилось на **401 наблюдение**, то есть на **41.7%**.

Дополнительно:

- **75 классов** — F1 улучшился;
- **53 класса** — без изменений;
- **19 классов** — получили снижение F1.

Это важно с инженерной точки зрения: improvement показал результат не только в агрегированной метрике, но и в фактической структуре ошибок.

Финальная визуализация строится в Step 20.1 как единая Before / After Error Map.

---

# Production decision

После inference и drift monitoring система сформировала эксплуатационное решение.

### Monitoring signals

- Numeric drift: **0.0230**
- Categorical drift: **0.0722**
- Confidence drift: **0.0136**
- Novelty signal: **0.1890**
- Target drift: **0.1020**
- Overall drift: **0.0579**
- Risk level: **LOW**

### Operational decision

```text
Decision:
NO_RETRAINING_REQUIRED

System action:
CONTINUE_PRODUCTION

Automatic model promotion:
BLOCKED
```

Это означает, что текущая модель может продолжать использоваться в production-контуре под действующими контролями мониторинга и безопасности.

При этом AutoFlow **не получает права самостоятельно заменить принятую модель** только на основании обнаруженного drift.

---

# Architecture

```text
RAW DATA
   ↓
Data Inventory & Data Contract
   ↓
EDA BEFORE
   ↓
AI-EDA / Data Quality
   ↓
Duplicate Forensics
   ↓
Canonical Dataset
   ↓
Temporal Split
   ↓
Feature Engineering
   ↓
Baseline ML
   ↓
Baseline Error Diagnostics
   ↓
Automated Model Improvement
   ↓
Accepted Model
   ↓
Intelligent Inference
   ↓
Confidence-based Routing
   ↓
Feedback & Drift Detection
   ↓
Automated Decision Orchestration
   ↓
Persistent State & Recovery
   ↓
Production Interface
   ↓
Final Model Audit
```

---

# Steps 1–4 — Data Discovery & Quality Control

На первом уровне система определяет:

- доступные вычислительные ресурсы;
- размер исходных данных;
- schema;
- типы признаков;
- пропуски;
- временные диапазоны;
- cardinality;
- target distribution;
- потенциальные leakage-признаки;
- дубликаты.

Для полного анализа используется DuckDB.

Выполняются:

- полный `COUNT(*)`;
- NULL profiling;
- numeric profiling;
- temporal profiling;
- target profiling;
- automated feature-role analysis.

### Duplicate Forensics

Дубликаты рассматриваются не как простая операция `drop_duplicates()`, а как отдельная инженерная задача.

Система сначала устанавливает природу повторений, а уже после этого формирует решение о canonicalization.

Результат:

```text
Source rows                 21,960,000
Unique canonical rows          209,999
Repeated rows               21,750,001
```

---

# Steps 5–10 — Data Preparation & Feature Pipeline

После data audit формируется Canonical Dataset.

```text
RAW
21,960,000 rows
        ↓
Forensic Deduplication
        ↓
Canonical Dataset
209,999 rows
```

Контролируются:

- data accounting;
- unique keys;
- target integrity;
- temporal integrity;
- schema consistency;
- feature roles;
- high-cardinality features;
- потенциальный leakage.

Canonical Dataset сохраняется в Parquet.

Основной принцип — тяжёлые операции выполнять resource-aware способом и не загружать исходный CSV целиком в Python RAM.

---

# Steps 11–13 — Canonical ML Foundation

На этом уровне формируется воспроизводимая ML-основа:

```text
Canonical Dataset
        ↓
Train / Validation / Test
        ↓
Feature Contract
        ↓
Canonical Baseline
        ↓
Reference Profile
        ↓
Error Diagnostics
```

Особое внимание уделяется **Zero Data Leakage**.

Для high-cardinality признаков используется memory-safe обработка, а для categorical space применяется hashing.

Canonical ML contract включает:

- 10 numeric features;
- 28 categorical features;
- 147 target classes;
- hash dimension `131072`;
- target `Complaint Type`;
- identifier `Unique Key`;
- temporal field `Created Date`.

Baseline обучается инкрементально батчами, что позволяет работать с canonical dataset без формирования огромной полной матрицы в Python RAM.

---

# Step 14 — Automated Model Improvement

Step 14 переводит canonical baseline в контролируемый improvement-контур.

Система формирует ограниченный набор диагностически обоснованных trial-конфигураций и сравнивает их **только на Validation**.

Проверяются:

- увеличение числа эпох;
- сглаженные class weights;
- совместное увеличение обучения и class weighting;
- регуляризация.

Наивная стратегия `class_weight="balanced"` не используется, поскольку экстремальные веса для редких классов могут ухудшать устойчивость классификатора.

### Acceptance flow

```text
Baseline
   ↓
Automated Trials
   ↓
Validation Comparison
   ↓
Best Trial
   ↓
Blind Test
   ↓
IMPROVED_MODEL_ACCEPTED
```

Принятая модель:

`SmoothedWeights_4Epochs_Reg3e5`

Она становится текущей production-кандидатурой для inference, routing и monitoring.

---

# Step 15 — Intelligent Inference & Automated Routing

Step 15 переводит принятую модель из режима оценки в эксплуатационный контур.

Система:

1. определяет конкретный принятый `model_run_id`;
2. проверяет входную schema и feature contract;
3. выполняет streaming inference батчами;
4. рассчитывает confidence и prediction margin;
5. автоматически маршрутизирует записи;
6. сохраняет checkpoint и execution trail.

### Routing

```text
Prediction
     ↓
Confidence / Margin
     ↓
┌─────────────┬────────────┬──────────────┐
↓             ↓            ↓
AUTO_ROUTE    REVIEW       HUMAN_REVIEW
```

Для текущего execution flow входной набор содержит **31,500 строк** и имеет **100% schema coverage**.

Ключевой принцип:

> **Человек не управляет каждым этапом обработки. Система выполняет рутинную работу самостоятельно и обращается к человеку только тогда, когда автоматического решения недостаточно.**

---

# Step 16 — Feedback & Drift Detection

Step 16 превращает inference-контур в self-monitoring system.

После обработки входящего потока система сравнивает текущие данные с Training Reference Profile и анализирует:

- numeric drift;
- categorical drift;
- confidence drift;
- novelty signal;
- target drift, если target доступен;
- overall drift risk.

Отсутствие target в новом production-файле не считается ошибкой.

Архитектурный принцип:

```text
DRIFT DETECTED
      ↓
DECISION
      ↓
RETRAINING REQUEST
```

а не:

```text
DRIFT DETECTED
      ↓
AUTOMATIC MODEL REPLACEMENT
```

---

# Step 17 — Automated Decision Orchestration

Step 17 является управляющим слоем AutoFlow AI.

Он объединяет результаты inference, routing и monitoring и формирует единое machine-readable решение.

Перед принятием решения система проверяет согласованность:

- `job_id`;
- `model_run_id`;
- selected trial;
- input fingerprint;
- completion status upstream-шагов.

### Decision policy

```text
HIGH risk
   ↓
RETRAINING_RECOMMENDED
   ↓
Create retraining request

MEDIUM risk
   ↓
REVIEW_RECOMMENDED
   ↓
Increase human review

LOW risk
   ↓
NO_RETRAINING_REQUIRED
   ↓
CONTINUE_PRODUCTION
```

Automatic model promotion остаётся заблокированным.

---

# Step 18 — Persistent State & Recovery

Step 18 формирует согласованный recovery artifact для production state.

В bundle сохраняются ключевые элементы состояния:

- current model pointer;
- model run metadata;
- execution state;
- checkpoints;
- manifests;
- decision state;
- artifact fingerprints.

Перед восстановлением система проверяет согласованность production state и связанных артефактов.

Важно: в текущем Kaggle-сценарии bundle создаётся в `/kaggle/working`, поэтому это **recovery artifact, подготовленный и проверенный на целостность**, а не доказанный permanent cross-session storage.

Для полноценной промышленной эксплуатации bundle должен быть передан во внешнее persistent storage.

---

# Step 19 — Production Interface

Step 19 завершает notebook-контур как production entrypoint:

```text
NEW INPUT
   ↓
VALIDATION
   ↓
PRODUCTION HANDOFF
```

Production interface автоматически ищет поддерживаемые входные форматы:

- CSV;
- Parquet;
- TXT.

Проверяются:

- формат;
- размер;
- SHA-256 fingerprint;
- наличие identifier;
- наличие temporal field;
- schema compatibility;
- наличие target, если он доступен.

### Важный пользовательский сценарий

Если в `autoflow_inbox` нет файла, система не завершается ошибкой.

Она переходит в штатное состояние:

`WAITING_FOR_INPUT`

Это позволяет использовать production interface как контролируемую точку входа для новых аналитических отчётов.

---

# Step 20 — Final Model Audit

Step 20 завершает модельный цикл и выполняет финальный независимый аудит принятой модели.

На этом этапе модель:

- не переобучается;
- не выбирается заново;
- не заменяется.

Вместо этого используется тот же Validation Dataset, который применялся для baseline diagnostics.

### Final Before → After

```text
BASELINE
   ↓
961 errors

IMPROVED / ACCEPTED MODEL
   ↓
560 errors
```

Главная задача Step 20 — показать, что improvement изменил не только агрегированную метрику, но и структуру классификационных ошибок.

Именно поэтому финальный аудит включает:

- Accuracy;
- Macro-F1;
- Macro-Recall;
- Weighted-F1;
- error count;
- Before / After confusion matrix;
- independent Blind Test.

---

# Ключевые инженерные принципы

- Resource-Aware Processing
- DuckDB Out-of-Core Execution
- Batch / Incremental Training
- Zero Data Leakage
- Temporal Validation
- Evidence-Based Model Improvement
- Feature Contract
- Model Contract
- Artifact Integrity
- SHA-256 Fingerprints
- Checkpoint / Resume
- Human-in-the-Loop Routing
- Drift Monitoring
- Controlled Model Promotion
- Reproducibility

---

# Technology Stack

Python · DuckDB · Pandas · NumPy · SciPy Sparse · scikit-learn · SGDClassifier · Logistic Regression · LightGBM · PyArrow / Parquet · joblib · psutil · Matplotlib · JSON · SHA-256 · Kaggle

---

# Notebook structure

Основной notebook содержит полный рабочий контур проекта:

| Step | Назначение |
|---|---|
| 1 | Инициализация проекта и аудит вычислительной среды |
| 2 | Data Inventory & Data Contract |
| 3 | EDA BEFORE |
| 4 | AI-EDA / Data Quality & Cleaning Strategy |
| 4.1 | Duplicate Forensics |
| 5–10 | Data Preparation, Feature Engineering и ML Foundation |
| 11 | AutoFlow Decision Engine |
| 12 | Resource-Aware ML Planner |
| 13 | Canonical Baseline ML + Visual Diagnostics |
| 14 | Automated Model Improvement |
| 15 | Intelligent Inference & Automated Routing |
| 16 | Feedback & Drift Detection |
| 17 | Automated Decision Orchestration |
| 18 | Persistent State & Recovery |
| 19 | Production Interface |
| 20 | Final Model Audit + Before / After Error Map + Final Project Result |

---

# Project status

**FINAL — ACCEPTED MODEL / ERROR STRUCTURE IMPROVED**

```text
Validation Macro-F1
0.750141 → 0.867885

Validation errors
961 → 560

Error reduction
401 (41.7%)

Risk
LOW

Decision
NO_RETRAINING_REQUIRED

System action
CONTINUE_PRODUCTION

Automatic promotion
BLOCKED
```

На текущем этапе модель принята для controlled production operation.

AutoFlow продолжает следить за состоянием данных и модели. При будущей деградации система должна не бесконтрольно заменить модель, а сформировать контролируемое решение о дальнейшем вмешательстве.

---

# Что демонстрирует проект

AutoFlow AI демонстрирует не только обучение multiclass-модели, а полный инженерный подход к автоматизации аналитического процесса:

```text
BIG DATA
   ↓
DATA QUALITY
   ↓
FORENSIC CLEANING
   ↓
LEAKAGE CONTROL
   ↓
MODEL TRAINING
   ↓
MODEL IMPROVEMENT
   ↓
INFERENCE
   ↓
HUMAN-IN-THE-LOOP ROUTING
   ↓
DRIFT MONITORING
   ↓
OPERATIONAL DECISION
   ↓
RECOVERY
   ↓
FINAL AUDIT
```

Главная идея проекта — **не просто получить хорошую модель, а убрать из ежедневной работы аналитика всю повторяющуюся техническую цепочку вокруг этой модели**.

В результате человек получает не ещё один сложный ML-инструмент, которым нужно управлять вручную, а контролируемый автоматизированный контур, который сам выполняет рутинную обработку, показывает сомнительные случаи и оставляет человеку именно те решения, где действительно требуется его участие.
