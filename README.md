# AutoFlow AI

### Autonomous Data-to-Decision Optimization Platform

> **От данных к решению — без ручного прохождения ML pipeline.**

[![Status](https://img.shields.io/badge/status-in--development-orange)]()
[![Python](https://img.shields.io/badge/Python-3.x-blue)]()
[![DuckDB](https://img.shields.io/badge/DuckDB-Big%20Data-yellow)]()
[![ML](https://img.shields.io/badge/ML-production--oriented-green)]()
[![Kaggle](https://img.shields.io/badge/Runtime-Kaggle-20BEFF)]()

---

## Проект

**AutoFlow AI** — production-oriented платформа для автоматизированной обработки больших потоков данных, интеллектуальной классификации и принятия эксплуатационных решений на основе состояния данных и модели.

* **21.96 млн записей**
* **46 признаков**
* **13.68 GiB на диске**

Проект построен не как отдельный ML-эксперимент, а как **сквозной autonomous data-to-decision pipeline**, в котором система самостоятельно проходит путь:

```text
RAW DATA
   ↓
Data Audit
   ↓
Quality Control
   ↓
Canonical Dataset
   ↓
Feature Pipeline
   ↓
Leakage Control
   ↓
Baseline / Model Selection
   ↓
Model Optimization
   ↓
Inference
   ↓
Confidence
   ↓
Automated Routing
   ↓
Drift Monitoring
   ↓
Decision
   ↓
[planned]
Orchestration
   ↓
Persistent State
   ↓
Production Interface
```

Основной сценарий проекта — автоматическая классификация обращений граждан на основе датасета **NYC 311 Service Requests**.

---

# Бизнес-задача

В традиционном ML workflow после обучения модели значительная часть работы остаётся ручной:

* подготовить новый файл;
* проверить его структуру;
* убедиться, что используется правильная версия модели;
* запустить inference;
* проверить confidence;
* определить, какие записи можно обработать автоматически;
* отправить сомнительные случаи человеку;
* проверить drift;
* решить, требуется ли переобучение;
* сохранить результаты запуска.

**AutoFlow AI автоматизирует именно этот эксплуатационный контур.**

Цель проекта — построить систему, которая после подготовки и принятия production-модели может принимать новый входящий отчёт и самостоятельно проходить путь:

```text
Input
→ Validation
→ Inference
→ Confidence
→ Routing
→ Monitoring
→ Decision
```

При этом система не должна автоматически заменять production-модель только потому, что detector обнаружил drift.

---

# SMART Goal

## Specific

Создать автономный pipeline для обработки больших наборов данных и production-oriented классификации обращений, который объединяет:

* Big Data processing;
* automated data quality control;
* feature engineering;
* leakage prevention;
* model benchmarking;
* model optimization;
* inference;
* confidence-based routing;
* drift monitoring;
* machine-readable operational decisions.

## Measurable

Платформа должна:

* обрабатывать исходный датасет объёмом **13.68 GiB**;
* работать с **21.96 млн записей**;
* выполнять полный data audit без необходимости загружать весь CSV в Pandas;
* формировать проверенный canonical dataset;
* автоматически выбирать и принимать модель;
* выполнять batch inference;
* разделять predictions на `AUTO_ROUTE`, `REVIEW` и `HUMAN_REVIEW`;
* контролировать numeric, categorical, confidence и novelty drift;
* формировать формальное решение о необходимости retraining;
* сохранять состояние выполнения и execution artifacts.

## Achievable

Для работы с большим объёмом данных используется resource-aware архитектура:

* DuckDB;
* out-of-core / streaming execution;
* adaptive memory budget;
* batch processing;
* Parquet;
* checkpointing;
* persistent artifacts;
* model registry;
* execution manifests.

Полный исходный CSV не загружается целиком в Python RAM. Тяжёлые операции выполняются непосредственно через DuckDB, а Pandas используется для компактных аналитических результатов.

## Relevant

Проект ориентирован не только на качество ML-модели, но и на задачу **автоматизации технического процесса**:

> модель должна быть частью работающей системы, а не конечной точкой ноутбука.

Поэтому архитектура включает контроль данных, воспроизводимость, registry, inference, routing, monitoring и recovery.

## Time-bound

Проект развивается по поэтапному production roadmap.

**Текущий статус: Steps 1–16 реализованы.**

Следующие этапы:

```text
16 — Feedback & Drift Detection
       ↓
17 — Automated Decision Orchestration
       ↓
18 — Persistent State & Recovery
       ↓
19 — Production Interface
       ↓
GitHub / Documentation / Demo
```

---

# Dataset

Проект использует открытый датасет:

**NYC 311 Service Requests**

Исходный набор:

| Parameter           |              Value |
| ------------------- | -----------------: |
| Records             |     **21,960,000** |
| Columns             |             **46** |
| CSV size            |      **13.68 GiB** |
| Target              | **Complaint Type** |
| Target classes      |            **152** |
| Numeric columns     |             **11** |
| Datetime columns    |              **3** |
| Categorical columns |             **32** |

Полный inventory и Data Contract формируются автоматически.

---

# Architecture

## Steps 1–4 — Data Discovery & Quality Control

На первом уровне система определяет:

* доступные вычислительные ресурсы;
* размер исходных данных;
* schema;
* типы признаков;
* пропуски;
* временные диапазоны;
* cardinality;
* target distribution;
* потенциальные leakage-признаки;
* дубликаты.

Для полного анализа используется DuckDB.

Выполняются:

* полный `COUNT(*)`;
* NULL profiling;
* numeric profiling;
* temporal profiling;
* target profiling;
* automated feature-role analysis.

### Duplicate Forensics

Отдельный этап анализирует дубликаты не как простое `drop_duplicates()`, а как отдельную инженерную задачу.

Система сначала определяет, содержит ли повторная запись новую информацию.

Результаты:

```text
Source rows                 21,960,000
Unique row fingerprints        209,999
Estimated duplicate rows     21,750,001
Duplicate rate                    99.04%
```

Решение об удалении принимается после forensic analysis, а не только по совпадению строк.

---

# Steps 5–10 — Data Preparation & Feature Pipeline

После data audit формируется **Canonical Dataset**.

```text
RAW
21,960,000 rows
        ↓
Duplicate Forensics
        ↓
Canonical Dataset
209,999 rows
```

Контролируются:

* data accounting;
* unique keys;
* target integrity;
* temporal integrity;
* schema consistency;
* feature roles;
* high-cardinality features;
* потенциальный leakage.

Canonical dataset сохраняется в Parquet.

Результат текущего pipeline:

```text
RAW rows                 21,960,000
Canonical rows              209,999
Removed rows             21,750,001
Duplicate rate                99.04%
Columns before                    46
Columns after                     45
```

---

# Steps 11–13 — Canonical Dataset → Baseline → Reference Profile

На этом уровне формируется воспроизводимая ML-основа:

```text
Canonical Dataset
        ↓
Train / Validation / Test
        ↓
Feature Contract
        ↓
Baseline
        ↓
Reference Profile
```

Особое внимание уделяется **zero data leakage**.

Для high-cardinality признаков используется train-only frequency encoding.

Baseline dataset:

```text
Train       147,000
Validation   31,500
Test         31,499
```

Размерность:

```text
Original ML features : 91
After preprocessing  : 963
```

Контроль целостности:

```text
Leakage check           PASSED
Feature schema alignment PASSED
```

---

# Step 14 — Automated Model Optimization

Step 14 переводит pipeline от baseline к автоматизированному поиску улучшенной модели.

Система сохраняет:

* model artifacts;
* registry;
* selected trial;
* metrics;
* manifest;
* model contract.

Принятая модель:

```text
SmoothedWeights_4Epochs_Reg3e5
```

Результаты:

```text
Validation Macro-F1 : 0.868468
Blind Test Macro-F1 : 0.820117
```

Модель принимается как конкретный versioned artifact, связанный с registry и manifest.

---

# Step 15 — Intelligent Inference & Automated Routing

Step 15 — переход от ML training к operational execution.

Система:

1. находит текущую принятую модель;
2. проверяет registry и manifest;
3. валидирует входную schema;
4. выполняет streaming inference;
5. рассчитывает confidence;
6. рассчитывает confidence второго кандидата;
7. рассчитывает prediction margin;
8. автоматически маршрутизирует записи.

## Routing

```text
                 Prediction
                     ↓
              Confidence / Margin
                     ↓
        ┌────────────┼────────────┐
        ↓            ↓            ↓
   AUTO_ROUTE      REVIEW    HUMAN_REVIEW
```

### AUTO_ROUTE

Модель достаточно уверена — запись может быть обработана автоматически.

### REVIEW

Пограничный случай, требующий дополнительной проверки.

### HUMAN_REVIEW

Уверенности недостаточно — решение передаётся человеку.

Пороговая политика калибруется только на Validation.

## Checkpointing

Step 15 сохраняет checkpoint после каждого batch.

Если выполнение прерывается, система может продолжить работу с последней сохранённой точки, не повторяя уже завершённые батчи.

---

# Step 16 — Feedback & Drift Detection

Step 16 превращает inference pipeline в **self-monitoring system**.

После обработки входящего потока система проверяет:

* numeric drift;
* categorical drift;
* confidence drift;
* novelty signal;
* target drift, если target доступен;
* overall drift risk.

При отсутствии target система не считает это ошибкой.

Production-файл может поступать без `Complaint Type`:

```text
daily_report.csv
```

В этом случае система анализирует:

```text
Covariate Drift
Confidence
Novelty
```

без искусственного вычисления target drift.

## Production Principle

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

Step 16 формирует machine-readable решение:

```text
NO_RETRAINING_REQUIRED
```

или:

```text
RETRAINING_RECOMMENDED
```

и **не заменяет production-модель самостоятельно**.

---

# Roadmap

## Step 17 — Automated Decision Orchestration

**Planned**

Step 17 станет orchestration layer между monitoring и дальнейшими действиями.

```text
NEW REPORT
    ↓
STEP 15
Inference + Routing
    ↓
STEP 16
Monitoring
    ↓
STEP 17
Decision Orchestration
    ↓
┌─────────────────────────────┐
│                             │
↓                             ↓
LOW / MEDIUM                 HIGH
│                             │
↓                             ↓
CONTINUE                 RETRAINING REQUEST
│                             │
↓                             ↓
SAVE RESULT              HUMAN / TRAINING QUEUE
```

Предполагаемые решения:

```text
NO_RETRAINING_REQUIRED
REVIEW_RECOMMENDED
RETRAINING_RECOMMENDED
```

Step 17 не будет самостоятельно переобучать и заменять production-модель.

Его задача — **принять решение и сформировать следующий machine-readable action**.

---

# Step 18 — Persistent State & Recovery

**Planned**

Цель — сделать pipeline устойчивым к перезапускам среды.

Вместо:

```text
Kaggle restart
    ↓
run 11
run 12
run 13
run 14
...
```

должно быть:

```text
New session
    ↓
Restore state
    ↓
Load accepted model
    ↓
Load last job
    ↓
Continue / start new job
```

Основой станет persistent state, содержащий:

* current model;
* model_run_id;
* last accepted run;
* job state;
* checkpoints;
* execution manifest;
* decision state.

---

# Step 19 — Production Interface

**Planned**

Финальный пользовательский сценарий должен быть максимально простым.

## Input

```text
Upload daily_report.csv
```

## AutoFlow

```text
Schema validation
        ↓
Production inference
        ↓
Routing
        ↓
Drift monitoring
        ↓
Decision
```

## Output

```text
Processed: 21,960,000

AUTO_ROUTE: ...
REVIEW: ...
HUMAN_REVIEW: ...

Drift: LOW

Decision:
NO_RETRAINING_REQUIRED
```

Пользователь работает не с внутренними ML-этапами, а с **единым operational entry point**.

---

# Engineering Principles

## Resource-Aware Processing

Система учитывает доступные CPU/RAM и не предполагает, что весь dataset можно безопасно загрузить в Pandas.

## Zero Data Leakage

Train, Validation и Test изолированы.

Feature engineering и calibration не используют Test для принятия решений.

## Evidence-Based Decisions

Инженерные решения подтверждаются измерениями:

* RAM;
* execution time;
* throughput;
* data quality;
* model metrics;
* drift signals.

## Reproducibility

Каждый значимый запуск связан с:

```text
run_id
model_run_id
job_id
manifest
registry
checkpoint
```

## Versioned Artifacts

Production execution использует конкретный принятый model artifact, а не случайно найденную «последнюю модель».

## Human-in-the-Loop

Автоматизация не означает отказ от человека.

```text
HIGH CONFIDENCE
→ AUTO_ROUTE

BORDERLINE
→ REVIEW

LOW CONFIDENCE
→ HUMAN_REVIEW
```

Человек подключается там, где автоматическое решение недостаточно надёжно.

---

# Technology Stack

## Data Engineering

* Python
* DuckDB
* Pandas
* NumPy
* Parquet
* SQL
* batch / streaming processing

## Machine Learning

* scikit-learn
* Logistic Regression
* LightGBM
* feature engineering
* frequency encoding
* model benchmarking
* confidence calibration

## MLOps / Production Engineering

* model registry
* model contracts
* data contracts
* run manifests
* checkpoints
* state management
* artifact validation
* drift detection
* automated routing

## Environment

* Kaggle
* Python runtime
* resource monitoring

---

# Current Project Status

| Stage                                                          | Status    |
| -------------------------------------------------------------- | --------- |
| Step 1 — Environment & Resource Audit                          | Completed |
| Step 2 — Data Inventory & Data Contract                        | Completed |
| Step 3 — EDA                                                   | Completed |
| Step 4 — Data Quality & Cleaning Strategy                      | Completed |
| Step 4.1 — Duplicate Forensics                                 | Completed |
| Steps 5–10 — Data Preparation & Feature Pipeline               | Completed |
| Steps 11–13 — Canonical Dataset / Baseline / Reference Profile | Completed |
| Step 14 — Automated Model Optimization                         | Completed |
| Step 15 — Intelligent Inference & Routing                      | Completed |
| Step 16 — Feedback & Drift Detection                           | Completed |
| Step 17 — Decision Orchestration                               | Planned   |
| Step 18 — Persistent State & Recovery                          | Planned   |
| Step 19 — Production Interface                                 | Planned   |
| GitHub Demo / Final Packaging                                  | Planned   |

---

# What makes AutoFlow AI different from a standard ML project?

Большинство учебных ML-проектов заканчиваются здесь:

```text
Dataset
   ↓
EDA
   ↓
Training
   ↓
Metric
   ↓
Model
```

AutoFlow AI продолжает этот путь:

```text
Dataset
   ↓
Data Quality
   ↓
Canonical Dataset
   ↓
Feature Pipeline
   ↓
Leakage Control
   ↓
Model
   ↓
Registry
   ↓
Inference
   ↓
Confidence
   ↓
Routing
   ↓
Drift Monitoring
   ↓
Decision
   ↓
Recovery
   ↓
Production Interface
```

Главный объект проектирования здесь — не только модель.

**Главный объект — автоматизированный процесс принятия решений вокруг модели.**

---

# Expected Final Result

После завершения Steps 17–19 AutoFlow AI должен поддерживать следующий сценарий:

```text
                    DAILY_REPORT.CSV
                           │
                           ▼
                 ┌───────────────────┐
                 │ Schema Validation │
                 └─────────┬─────────┘
                           │
                           ▼
                 ┌───────────────────┐
                 │ Production Model  │
                 └─────────┬─────────┘
                           │
                           ▼
                 ┌───────────────────┐
                 │    Inference      │
                 └─────────┬─────────┘
                           │
                           ▼
                 ┌───────────────────┐
                 │ Confidence +      │
                 │ Routing            │
                 └─────────┬─────────┘
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
        AUTO_ROUTE       REVIEW      HUMAN_REVIEW
                           │
                           ▼
                 ┌───────────────────┐
                 │ Drift Monitoring  │
                 └─────────┬─────────┘
                           │
                           ▼
                 ┌───────────────────┐
                 │ Decision          │
                 └─────────┬─────────┘
                           │
              ┌────────────┴────────────┐
              ▼                         ▼
       CONTINUE                  RETRAINING REQUEST
```

---

# Project Author

**Yuliya Prokhorova**

Machine Learning / Data Science

Проект разработан как практическая демонстрация навыков:

* Python development;
* Data Engineering;
* Big Data processing;
* Machine Learning;
* ML pipeline design;
* automation;
* MLOps principles;
* production-oriented system design.

---

# Project Status

**Active development**

Current milestone:

> **STEP 16 — Feedback & Drift Detection completed**

Next milestone:

> **STEP 17 — Automated Decision Orchestration**

Final target:

> **STEP 19 — Production Interface**

После завершения эксплуатационного контура проект будет представлен как воспроизводимая production-oriented демонстрация полного цикла:

**Data → ML → Inference → Automation → Monitoring → Decision**
