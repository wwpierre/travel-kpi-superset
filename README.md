# Travel KPI (Superset)

Интерактивный дашборд для анализа тревел-бронирований: **Bookings, Revenue, Cancel Rate**, каналы, направления, сезонность и сегменты.  
Собрано в **Apache Superset**. Данные — **синтетические** (safe for portfolio).

<img src="images/overview.png" width="900" alt="Overview screenshot"/>

---

## ✨ Что внутри

**KPI (L12M):**
- Bookings — количество бронирований  
- Revenue — выручка (только завершённые)  
- Cancel rate — доля отмен  
- Avg check — средний чек завершённых

**Чарты:**
- **Revenue & Cancel rate (Monthly)** — dual-axis line  
- **Bookings by Channel (Monthly)** — stacked area  
- **Top Destinations by Revenue** — bar (Top-10)  
- **Cancel Rate vs Lead Time** — line  
- **Seasonality Heatmap** — месяц × направление (выручка)  
- **Region × Channel — Revenue** — pivot-таблица с тоталами

**Фильтры (Apply to all):** Time range, Region, Channel, Customer type.  
**Tabs (рекомендуется):** `Обзор` (KPI+тренды), `Сегменты` (Heatmap+Pivot), `Риски` (Cancel).

---

## 🧰 Стек
- **Apache Superset** ≥ 5.x  
- **PostgreSQL** 14 (для примеров)  
- CSV/SQL — загрузка демо-датасета

---

## 🗃️ Структура репозитория

```
travel-kpi-superset/
├─ README.md
├─ images/                     # скриншоты для README
│  ├─ overview.png
│  ├─ segments.png
│  └─ destinations.png
├─ superset/
│  └─ Travel_KPI_export.zip    # экспорт дашборда/чартов из Superset
├─ data/
│  └─ bookings.csv             # синтетический датасет (если используешь CSV)
└─ sql/
   ├─ create_tables.sql
   └─ seed.sql                 # опционально (вьюхи/агрегаты)
```

---

## 📄 Схема данных (`public.bookings`)

```sql
CREATE TABLE public.bookings (
  booking_id       BIGINT PRIMARY KEY,
  user_id          BIGINT,
  booking_date     DATE,
  travel_date      DATE,
  destination      TEXT,
  region           TEXT,          -- APAC / Americas / EU / MEA
  user_home_country TEXT,
  channel          TEXT,          -- Web / MobileApp / Agency
  device           TEXT,          -- Desktop / Mobile
  customer_type    TEXT,          -- Leisure / Business
  promo_applied    BOOLEAN,
  lead_time_days   INT,           -- дни от бронирования до поездки
  nights           INT,
  adults           INT,
  children         INT,
  price            NUMERIC(12,2), -- валюта условная
  booking_status   TEXT           -- Completed / Cancelled
);
```

**Ключевые метрики (формулы):**
- **Revenue**  
  ```sql
  SUM(CASE WHEN booking_status='Completed' THEN price ELSE 0 END)
  ```
- **Bookings**  
  ```sql
  COUNT(*)
  ```
- **Cancel rate**  
  ```sql
  AVG(CASE WHEN booking_status='Cancelled' THEN 1.0 ELSE 0.0 END)
  ```
- **Avg check**  
  ```sql
  AVG(CASE WHEN booking_status='Completed' THEN price END)
  ```

---

## 🚀 Быстрый старт (локально)

> Требуется Docker.

1. **Клонируй репозиторий**
   ```bash
   git clone https://github.com/yourname/travel-kpi-superset.git
   cd travel-kpi-superset
   ```

2. **Подними PostgreSQL с демо-БД**
   ```bash
   docker run --name demo_pg --restart unless-stopped      -e POSTGRES_PASSWORD=postgres -e POSTGRES_USER=postgres -e POSTGRES_DB=demo      -p 5432:5432 -d postgres:14
   ```

3. **Создай таблицу и загрузи данные**
   ```bash
   # создать таблицу
   docker exec -i demo_pg psql -U postgres -d demo < sql/create_tables.sql

   # загрузить CSV в БД
   docker cp data/bookings.csv demo_pg:/tmp/bookings.csv
   docker exec -it demo_pg psql -U postgres -d demo      -c "\copy public.bookings FROM '/tmp/bookings.csv' CSV HEADER;"
   ```

4. **Открой Superset** (используй свой инстанс или Docker-стек Superset)
   - В Superset **Databases → + Database → PostgreSQL**  
     **SQLAlchemy URI:**
     ```
     postgresql+psycopg2://postgres:postgres@host.docker.internal:5432/demo
     ```
     (На Linux вместо `host.docker.internal` укажи IP хоста, например `127.0.0.1`.)

5. **Импортируй дашборд**
   - **Settings → Import dashboards** → `superset/Travel_KPI_export.zip`
   - Убедись, что датасеты привязаны к твоей БД `demo`.

Готово — дашборд появится в списке Dashboards.

---

## 📊 Как пользоваться

- Выбери период (по умолчанию — **Last 12 months**).  
- Отфильтруй по **Region / Channel / Customer Type**.  
- На вкладке **Обзор** смотри динамику KPI; на **Сегментах** — сезонность и разрезы; на **Рисках** — влияние lead-time на отмены.  
- Все графики синхронизированы фильтрами.

---

## 💡 Примеры инсайтов (на синтетических данных)

- Выручка концентрируется в **MobileApp** и **Web**, а **Agency** даёт меньшую долю.
- **Cancel rate** растёт при **lead_time > 45 дней** — риск «забытых» бронирований.
- **Top-5 направлений** дают львиную долю выручки; акции (**promo_applied**) заметно ускоряют бронирования в низкий сезон.
- Регионально **Americas** и **EU** стабильнее по сезонности, **APAC** — более волатилен.

---

## 📎 Лицензия

MIT. Данные синтетические и опубликованы исключительно для демонстрации навыков аналитики/визуализации.

---

