# nslookup
program Otomasi Jaringan
import socket
import subprocess
import datetime
import time
import os
from colorama import Fore, Style, init

# Inisialisasi warna terminal
init(autoreset=True)

# Nama file untuk menyimpan riwayat pencarian
LOG_FILE = "riwayat_nslookup.txt"

def clear():
    os.system('clear' if os.name == 'posix' else 'cls')

def garis():
    print(Fore.CYAN + "═" * 60)

def loading_animation(text="Mencari informasi DNS"):
    for i in range(3):
        print(Fore.YELLOW + f"{text}" + "." * (i+1), end="\r")
        time.sleep(0.4)
    print(" " * 50, end="\r")

def get_ttl(domain):
    """Ambil TTL domain via dig"""
    try:
        result = subprocess.run(["dig", "+noall", "+answer", domain],
                                capture_output=True, text=True)
        if result.stdout.strip():
            ttl_values = [line.split()[1] for line in result.stdout.split("\n") if line.strip()]
            return ttl_values[0] if ttl_values else "Tidak diketahui"
        return "Tidak ditemukan"
    except Exception:
        return "Tidak tersedia"

def get_record_types(domain):
    """Ambil tipe record DNS"""
    record_types = []
    for rtype in ["A", "AAAA", "MX", "CNAME", "NS"]:
        result = subprocess.run(["dig", "+short", domain, rtype],
                                capture_output=True, text=True)
        if result.stdout.strip():
            record_types.append(rtype)
    return record_types if record_types else ["Tidak ditemukan"]

def show_banner():
    """Menampilkan banner program"""
    clear()
    print(Fore.GREEN + "╔════════════════════════════════════════════════╗")
    print(Fore.GREEN + "║     🔍 PROGRAM DNS NSLOOKUP DEVASC + HISTORY    ║")
    print(Fore.GREEN + "╚════════════════════════════════════════════════╝\n")

def simpan_riwayat(teks):
    """Simpan hasil lookup ke file riwayat"""
    with open(LOG_FILE, "a") as f:
        f.write(teks + "\n")

def tampilkan_riwayat():
    """Menampilkan isi riwayat pencarian"""
    clear()
    print(Fore.CYAN + "╔════════════════════════════════════════════════╗")
    print(Fore.CYAN + "║              📜 RIWAYAT PENCARIAN DNS          ║")
    print(Fore.CYAN + "╚════════════════════════════════════════════════╝")
    garis()

    if not os.path.exists(LOG_FILE) or os.stat(LOG_FILE).st_size == 0:
        print(Fore.YELLOW + "Belum ada riwayat pencarian tersimpan.")
    else:
        with open(LOG_FILE, "r") as f:
            print(Fore.WHITE + f.read())
    garis()
    input(Fore.BLUE + "\nTekan ENTER untuk kembali ke menu...")

def nslookup(domain):
    """Lookup domain → IP"""
    waktu = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    start = time.time()
    garis()
    print(Fore.GREEN + f" Target : {domain}")
    print(Fore.MAGENTA + f" Waktu  : {waktu}")
    garis()

    loading_animation()

    try:
        host, alias, ip_addr = socket.gethostbyname_ex(domain)
        ttl = get_ttl(domain)
        record_types = get_record_types(domain)
        end = time.time()
        durasi = round(end - start, 3)

        hasil = (
            f"\n[INFO] Hasil NSLOOKUP:\n"
            f" Hostname Resmi : {host}\n"
            f" Alias          : {', '.join(alias) if alias else 'Tidak ada alias'}\n"
            f" Alamat IP      : {', '.join(ip_addr)}\n"
            f" TTL            : {ttl}\n"
            f" Record Tersedia: {', '.join(record_types)}\n"
            f" Durasi Lookup  : {durasi} detik\n"
        )

        print(Fore.YELLOW + hasil)
        print(Fore.GREEN + "[OK] Lookup berhasil!")
        garis()

        # Simpan ke riwayat
        simpan_riwayat(f"[{waktu}] Lookup Domain: {domain}\n{hasil}")

    except socket.gaierror:
        print(Fore.RED + f"[ERROR] Domain '{domain}' tidak ditemukan.")
    except Exception as e:
        print(Fore.RED + f"[ERROR] Terjadi kesalahan: {e}")

def reverse_lookup(ip):
    """Reverse lookup IP → domain"""
    waktu = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    start = time.time()
    garis()
    print(Fore.GREEN + f" Target IP : {ip}")
    print(Fore.MAGENTA + f" Waktu     : {waktu}")
    garis()

    loading_animation("Melakukan reverse lookup")

    try:
        host, alias, ip_addr = socket.gethostbyaddr(ip)
        end = time.time()
        durasi = round(end - start, 3)

        hasil = (
            f"\n[INFO] Hasil Reverse Lookup:\n"
            f" Hostname : {host}\n"
            f" Alias    : {', '.join(alias) if alias else 'Tidak ada alias'}\n"
            f" Alamat IP: {', '.join(ip_addr)}\n"
            f" Durasi Lookup: {durasi} detik\n"
        )

        print(Fore.YELLOW + hasil)
        print(Fore.GREEN + "[OK] Reverse lookup berhasil!")
        garis()

        # Simpan ke riwayat
        simpan_riwayat(f"[{waktu}] Reverse Lookup: {ip}\n{hasil}")

    except socket.herror:
        print(Fore.RED + f"[ERROR] Tidak dapat menemukan hostname untuk IP {ip}.")
    except Exception as e:
        print(Fore.RED + f"[ERROR] Terjadi kesalahan: {e}")

def main():
    """Menu utama"""
    while True:
        show_banner()
        print(Fore.YELLOW + "[1] Lookup Domain → IP (Masukkan domain manual)")
        print(Fore.YELLOW + "[2] Reverse Lookup (Masukkan IP manual)")
        print(Fore.YELLOW + "[3] Keluar")
        print(Fore.YELLOW + "[4] Tampilkan Riwayat Pencarian\n")

        pilihan = input(Fore.CYAN + "Pilih menu (1/2/3/4): ")

        if pilihan == "1":
            domain = input(Fore.YELLOW + "\nMasukkan nama domain (contoh: www.google.com): ").strip()
            nslookup(domain)
        elif pilihan == "2":
            ip = input(Fore.YELLOW + "\nMasukkan alamat IP (contoh: 8.8.8.8): ").strip()
            reverse_lookup(ip)
        elif pilihan == "3":
            print(Fore.GREEN + "\nTerima kasih telah menggunakan NSLOOKUP DEVASC + HISTORY!")
            break
        elif pilihan == "4":
            tampilkan_riwayat()
        else:
            print(Fore.RED + "\n[!] Pilihan tidak valid.")
            time.sleep(1)

        input(Fore.BLUE + "\nTekan ENTER untuk kembali ke menu...")

if __name__ == "__main__":
    main()
