import time
import random
from datetime import datetime
from legacy_order_service import LegacyOrderService, Order, OrderItem, Client

def generate_orders(count: int):
    rnd = random.Random(42)
    orders = []
    for i in range(1, count + 1):
        items_count = rnd.randint(1, 3)
        items = [
            OrderItem(
                id=k + 1,
                sku=f"SKU-{(i * 7 + k) % 150}",  # контролируемые дубликаты
                title=f"Item {k}",
                price=float(rnd.randint(100, 5000)),
                quantity=rnd.randint(1, 5)
            )
            for k in range(items_count)
        ]
        orders.append(
            Order(
                id=i,
                client_id=rnd.randint(1, 500),
                status=rnd.randint(1, 3),
                created_at=datetime.now(),
                items=items
            )
        )
    return orders

def main():
    print("=" * 70)
    print("  БЕНЧМАРК ЛАБОРАТОРНОГО ПРАКТИКУМА: ПРОФИЛИРОВАНИЕ И РЕФАКТОРИНГ (Python)")
    print("=" * 70 + "\n")

    service = LegacyOrderService()

    clients = [
        Client(id=i, name=f"Client_{i}", loyalty_tier=(i % 3) + 1)
        for i in range(1, 501)
    ]

    print("--- ПРОГРЕВ (WARMUP) ---")
    warmup = generate_orders(100)
    service.find_duplicate_skus(warmup)
    service.apply_discounts(warmup, clients)
    service.generate_batch_report(warmup)
    print("Прогрев завершен.\n")

    batch_sizes = [1000, 3000, 5000, 10000]

    print(f"{'N (Заказов)':<12} | {'Поиск дубликатов':<18} | {'Расчет скидок':<18} | {'Генерация отчета':<18} | {'ОБЩЕЕ ВРЕМЯ':<15}")
    print("-" * 88)

    for n in batch_sizes:
        orders = generate_orders(n)

        # Замер 1: Поиск дубликатов
        t0 = time.perf_counter()
        service.find_duplicate_skus(orders)
        t_dup = (time.perf_counter() - t0) * 1000.0

        # Замер 2: Скидки
        t0 = time.perf_counter()
        service.apply_discounts(orders, clients)
        t_disc = (time.perf_counter() - t0) * 1000.0

        # Замер 3: Отчет
        t0 = time.perf_counter()
        service.generate_batch_report(orders)
        t_rep = (time.perf_counter() - t0) * 1000.0

        t_total = t_dup + t_disc + t_rep

        print(f"{n:<12} | {t_dup:15.2f} мс | {t_disc:15.2f} мс | {t_rep:15.2f} мс | {t_total:12.2f} мс")

    print("\nЗамеры завершены. Перенесите данные в отчет templates/REPORT_TEMPLATE.md.")

if __name__ == '__main__':
    main()
