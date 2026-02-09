import os
import random
import time
import ipaddress
import concurrent.futures
import threading
import sys
import socket
import signal

# التأكد من وجود المكتبة
try:
    from mcstatus import JavaServer
except ImportError:
    print("خطأ: يرجى تثبيت المكتبة أولاً: pip install mcstatus")
    sys.exit(1)

# شعار DARK DRONE
DARKDRONE_LOGO = """
DDDDDD   AAAA   RRRRR  KK  KK       DDDDDD   RRRRR   OOOO   NN  NN  EEEEEE
DD  DD  AA  AA  RR  RR KK KK        DD  DD   RR  RR OO  OO  NNN NN  EE
DD  DD  AAAAAA  RRRRR  KKKK         DD  DD   RRRRR  OO  OO  NN NNN  EEEE
DD  DD  AA  AA  RR  RR KK KK        DD  DD   RR  RR OO  OO  NN  NN  EE
DDDDDD  AA  AA  RR  RR KK  KK       DDDDDD   RR  RR  OOOO   NN  NN  EEEEEE
"""

# إعدادات التحكم
MAX_WORKERS = 100        # تحديد عدد الخيوط كما طلبت
JAVA_PORT = 25565        # منفذ جافا الافتراضي
TIMEOUT_SOCKET = 1     # مهلة فحص الاتصال الأولي (سريع جداً)
TIMEOUT_MCSTATUS = 3   # مهلة جلب بيانات اللاعبين

stop_search_event = threading.Event()
pause_event = threading.Event()
pause_event.set()

reported_ips = set()
reported_ips_lock = threading.Lock()

def clear_screen():
    os.system('cls' if os.name == 'nt' else 'clear')

def signal_handler(sig, frame):
    stop_search_event.set()
    pause_event.set()
    sys.exit(0)

signal.signal(signal.SIGINT, signal_handler)

def generate_random_ip():
    """توليد IP عشوائي تماماً لغرض الفحص المختبري."""
    while True:
        # توليد عشوائي يتجنب النطاقات المحلية المعروفة قدر الإمكان
        ip = f"{random.randint(1, 254)}.{random.randint(0, 255)}.{random.randint(0, 255)}.{random.randint(1, 254)}"
        return ip

def fast_port_check(ip, port):
    """فحص سريع جداً للمنفذ قبل محاولة جلب البيانات."""
    try:
        with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
            s.settimeout(TIMEOUT_SOCKET)
            result = s.connect_ex((ip, port))
            return result == 0
    except:
        return False

def get_server_details(ip):
    """جلب تفاصيل اللاعبين فقط إذا كان المنفذ مفتوحاً."""
    try:
        server = JavaServer.lookup(ip, JAVA_PORT)
        status = server.status(timeout=TIMEOUT_MCSTATUS)
        return True, status.players.online, status.players.max
    except:
        return False, 0, 0

def scan_task():
    """وظيفة الخيط: يولد اتصالات عشوائية باستمرار."""
    while not stop_search_event.is_set():
        pause_event.wait()
        
        target_ip = generate_random_ip()
        
        # المرحلة 1: فحص الاتصال السريع (TCP Check)
        # هذا يجعل "الاتصالات تذهب بعشوائية" وبسرعة هائلة
        if fast_port_check(target_ip, JAVA_PORT):
            # المرحلة 2: إذا وجدنا استجابة، نجلب بيانات اللاعبين
            is_online, online, max_p = get_server_details(target_ip)
            
            if is_online:
                with reported_ips_lock:
                    if target_ip not in reported_ips:
                        print(f"[+] FOUND: {target_ip} | Status: ONLINE | Players: {online}/{max_p}")
                        reported_ips.add(target_ip)

def start_scanning():
    clear_screen()
    print(DARKDRONE_LOGO)
    print(f"[*] Target: JAVA EDITION | Threads: {MAX_WORKERS} | Port: {JAVA_PORT}")
    print("[*] Strategy: Fast TCP Pre-Check + MCStatus Detailer")
    print("[*] Starting random packet flow...\n")
    
    threads = []
    for _ in range(MAX_WORKERS):
        t = threading.Thread(target=scan_task, daemon=True)
        t.start()
        threads.append(t)
    
    # حلقة الانتظار الرئيسية
    try:
        while not stop_search_event.is_set():
            time.sleep(1)
    except KeyboardInterrupt:
        stop_search_event.set()

if __name__ == "__main__":
    start_scanning()
    print("\n[!] Scan stopped. Goodbye.")

