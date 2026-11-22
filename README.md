# Restaurant Management API

A Django backend for restaurant menu, orders, tables, reservations, and inventory. It uses SQLite by default and includes the Django admin interface.

## Setup

Requires Python 3.12 or newer.

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
python manage.py migrate
python manage.py createsuperuser
python manage.py runserver
```

Open the admin panel at `http://127.0.0.1:8000/admin/`. Add menu items, inventory stock, and tables there. Django admin staff can also use protected inventory and reporting endpoints with their logged-in session.

Set `DJANGO_SECRET_KEY`, `DJANGO_DEBUG=false`, and `DJANGO_ALLOWED_HOSTS` in a production environment. The default secret key and SQLite configuration are for local development.

## API

All responses use JSON. Reservation dates must be ISO 8601 datetimes with a timezone offset. Currency values are returned as decimal strings.

| Method | Endpoint | Access | Purpose |
| --- | --- | --- | --- |
| GET | `/api/health/` | Public | Health check |
| GET | `/api/menu/` | Public | List available menu items |
| POST | `/api/orders/` | Public | Place an order and decrement stock atomically |
| GET, PATCH | `/api/orders/{id}/` | Staff | View an order or advance/cancel its status |
| GET | `/api/tables/availability/?party_size=2&start_at=...&duration_minutes=120` | Public | List suitable tables without overlapping reservations |
| POST | `/api/reservations/` | Public | Reserve the smallest suitable available table |
| GET | `/api/inventory/` | Staff | View stock |
| PATCH | `/api/inventory/{menu_item_id}/` | Staff | Update `quantity` and/or `low_stock_threshold` |
| GET | `/api/reports/daily-sales/?date=YYYY-MM-DD` | Staff | Completed orders and sales for a date |
| GET | `/api/reports/stock-alerts/` | Staff | Inventory at or below its alert threshold |

Example order body:

```json
{
  "table_id": 1,
  "items": [
    {"menu_item_id": 1, "quantity": 2}
  ]
}
```

Order statuses progress from `placed` to `preparing`, `ready`, and `completed`. Staff may cancel an active order; cancellation restores its inventory exactly once. Staff-only endpoints use Django session authentication, so log in through `/admin/` in the same browser session.

## Tests

```powershell
python manage.py test
```
