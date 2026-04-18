# ScanBLE
<details>
<summary>Scan for BLE devices within range</summary>

```python
import asyncio
from bleak import BleakScanner
import time
import sys

RED = '\033[91m'
GREEN = '\033[92m'
RESET = '\033[0m'

async def scan_with_history():
    print("Сканирование BLE устройств")
    
    # Словарь для хранения устройств: {MAC: {"name": имя, "last_seen": время}}
    devices_history = {}
    
    try:
        while True:
            # Сканируем 2 секунды
            active_devices = await BleakScanner.discover(timeout=2)
            
            # Множество MAC-адресов активных устройств
            active_macs = set()
            
            # Обновляем историю
            for device in active_devices:
                active_macs.add(device.address)
                if device.address in devices_history:
                    devices_history[device.address]["last_seen"] = time.time()
                    if device.name and not devices_history[device.address]["name"]:
                        devices_history[device.address]["name"] = device.name
                else:
                    devices_history[device.address] = {
                        "name": device.name if device.name else "Unknown",
                        "last_seen": time.time()
                    }
            
            # Очищаем экран (один раз перед выводом)
            print("\033[2J\033[H", end="")
            
            # Заголовок
            print(f"=== BLE устройства [{time.strftime('%H:%M:%S')}] ===")
            print()
            
            # Выводим все устройства
            if devices_history:
                for mac, info in devices_history.items():
                    name = info["name"]
                    is_active = mac in active_macs
                    status = "+" if is_active else "-"
                    if is_active:
                        print(f"{GREEN}[{status}] {mac} - {name}{RESET}")
                    else:
                        print(f"[{status}] {mac} - {name}")
            else:
                print("Устройства не найдены")
            
            print()
            print("Нажмите Ctrl+C для выхода")
            
            # Небольшая задержка перед следующим сканированием
            await asyncio.sleep(1)
            
    except KeyboardInterrupt:
        print("\n\nОстановлено.")
        sys.exit(0)

if __name__ == "__main__":
    asyncio.run(scan_with_history())
</details>
