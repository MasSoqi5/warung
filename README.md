# warung

import csv
import os
import pwinput # menutup password dengan *
from prettytable import PrettyTable
from datetime import datetime

# Mendapatkan direktori tempat skrip saat ini berada
base_dir = os.path.dirname(os.path.abspath(__file__))
# Nama file CSV untuk menyimpan data user dan transaksi di folder yang sama dengan aplikasi
filename_user= os.path.join(base_dir, 'users.csv')
filename_menu = os.path.join(base_dir, 'menu.csv')

# Fungsi Quick Sort untuk mengurutkan data user berdasarkan kolom Nama
def quick_sort(arr):
    if len(arr) <= 1:
    
        return arr
    else:
        wadah = arr[len(arr) // 2]
        left = [x for x in arr if x[1] < wadah[1]]
        middle = [x for x in arr if x[1] == wadah[1]]
        right = [x for x in arr if x[1] > wadah[1]]
        return quick_sort(left) + middle + quick_sort(right)

#Fungsi clean screen
def clear_screen():
    os.system('cls' if os.name == 'nt' else 'clear')

# Fungsi untuk membuat file CSV jika belum ada
def buat_file_csv():
    if not os.path.exists(filename_user):
        with open(filename_user, mode='w', newline='') as file:
            writer = csv.writer(file)
            writer.writerow(["ID", "Nama", "Password", "Type","attempts","Duit"])
    if not os.path.exists(filename_menu):
        with open(filename_menu, mode='w', newline='') as file:
            writer = csv.writer(file)
            writer.writerow(["ID_menu", "waktu", "Type", "nama_menu", "Harga"])

# Fungsi untuk mengecek apakah ID user sudah ada
def cek_id_user(id_user):
    with open(filename_user, mode='r') as file:
        reader = csv.reader(file)
        for row in reader:
            if row[0] == id_user:
                return True
    return False

# Fungsi untuk mengecek apakah ID menu sudah ada
def cek_id_menu(id_menu):
    with open(filename_menu, mode='r') as file:
        reader = csv.reader(file)
        for row in reader:
            if row[0] == id_menu:
                return True
    return False

# Fungsi untuk menambah data user baru
def tambah_user(id_user, nama, password, type, attempts, Duit):
    if cek_id_user(id_user):
        print(f"\033[3mID pelanggan {id_user} sudah ada. Gunakan ID lain.\033[0m")  # Italic text
        return
    with open(filename_user, mode='a', newline='') as file:
        writer = csv.writer(file)
        writer.writerow([id_user, nama, password, type, attempts, Duit])
    print(f"User {nama} berhasil ditambahkan.")

# Fungsi untuk menambah data menu baru
def tambah_menu(ID_menu, waktu, Type,nama_menu,Harga):
    if cek_id_menu(ID_menu):
        print(f"\033[3mID MENU {ID_menu} sudah ada. Gunakan ID lain.\033[0m")  # Italic text
        return
    with open(filename_menu, mode='a', newline='') as file:
        writer = csv.writer(file)
        writer.writerow([ID_menu, waktu, Type, nama_menu, Harga])
    print(f"Menu {nama_menu} berhasil ditambahkan.")

# Fungsi untuk menyimpan data pengguna ke file CSV
def save_users(users):
    with open(filename_user, mode="w", newline="") as file:
        # Menyiapkan penulis CSV
        writer = csv.writer(file)
        # Menulis header ke dalam file1
        writer.writerow(["ID", "Nama", "Password", "Type", "attempts", "Duit"])
        # Menulis data untuk setiap user
        for ID, data in users.items():
            writer.writerow([
                ID, 
                data["Nama"], 
                data["Password"], 
                data["Type"], 
                data["attempts"], 
                data["Duit"]
            ])

# Fungsi untuk memastikan file CSV pengguna ada
def cek_fileuser():
    if not os.path.exists(filename_user):
        print(f"File {filename_user} tidak ditemukan. Membuat file dengan data default.")
        with open(filename_user, mode="w", newline="") as file:
            fieldnames = ["ID","Nama", "Password", "Type", "attempts", "Duit"]
            writer = csv.DictWriter(file, fieldnames=fieldnames)
            writer.writeheader()

# Fungsi untuk membaca data pengguna dari file CSV
def load_users():
    users = {}
    cek_fileuser()
    with open(filename_user, mode="r") as file:
        reader = csv.DictReader(file)
        for row in reader:
            users[row["ID"]] = {
                "Nama": row["Nama"],
                "Password": row["Password"],
                "Type": int(row["Type"]),
                "attempts": int(row["attempts"]),
                "Duit": float(row["Duit"])
            }
    return users

# Fungsi untuk membaca data menu dari file CSV
def load_menu(filename):
    menu_data = []
    try:
        with open(filename, newline='', encoding='utf-8') as csvfile:
            reader = csv.DictReader(csvfile)
            for row in reader:
                menu_data.append(row)
    except FileNotFoundError:
        print(f"File {filename} tidak ditemukan.")
    except Exception as e:
        print(f"Terjadi kesalahan: {e}")
    return menu_data

# Fungsi untuk login
def login():
    ADMIN = "ADMIN"  # ID untuk admin, bisa disesuaikan
    ADMIN_PASSWORD = "ADMIN123"  # Password admin
    users = load_users()  # Load data pengguna dari file CSV
    while True:
        ID = input("Masukkan ID: ")
        if ID == ADMIN:  # Mengecek apakah ID yang dimasukkan adalah admin
            password = pwinput.pwinput(prompt="Masukkan Password: ")
            #input("Masukkan password: ")
            if password == ADMIN_PASSWORD:
                clear_screen()
                print(f"Selamat datang, {ADMIN}!")
                return ADMIN, ADMIN, ADMIN  # Kembalikan ADMIN dan ADMIN sebagai tipe
            else:
                print("Password admin salah!")
                continue  # Kembali ke input ID jika password salah
        if ID in users:  # Mengecek apakah ID ada dalam data user
            user = users[ID]
            if user["attempts"] >= 3:
                print(f"Akun {ID} terkunci. Hubungi admin.")
                return None, None, None
            # Mengecek password untuk user yang sudah terdaftar
            for _ in range(3 - user["attempts"]):
                password = pwinput.pwinput(prompt="Masukkan Password: ")
                if password == user["Password"]:
                    if user["Type"] == 1:  # Jika user adalah member biasa
                        clear_screen()
                        print(f"Selamat datang, {user['Nama']}! Anda adalah member (1) 'BIASA'")
                    elif user["Type"]==2:  # Jika user adalah member VIP
                        clear_screen
                        print(f"Selamat datang, {user['Nama']}! Anda adalah member(2) 'VIP'")
                    return ID, user["Nama"], user["Type"]
                else:
                    print("Password salah!")
                    user["attempts"] += 1
                    save_users(users)  # Simpan perubahan ke file CSV
            
            if user["attempts"] >= 3:
                print(f"Akun {ID} terkunci. Hubungi admin.")
                save_users(users)  # Simpan perubahan ke file CSV
                return None, None, None
        else:
            print("User tidak ditemukan!")

# Fungsi untuk melihat semua data user dengan PrettyTable dan di sortir by nama
def lihat_data_user():
    with open(filename_user, mode='r') as file:
        reader = csv.reader(file)
        data = list(reader)
        if len(data) <= 1:
            print("Tidak ada data user.")
        else:
            sorted_data = quick_sort(data[1:])
            table = PrettyTable(["ID", "Nama", "Password", "Type", "attempts","Duit"])
            for row in sorted_data:
                table.add_row(row)
            print(table)

# Fungsi untuk melihat semua data menu dengan PrettyTable dan di sortir by nama
def lihat_data_menu():
    with open(filename_menu, mode='r') as file:
        reader = csv.reader(file)
        data = list(reader)
        if len(data) <= 1:
            print("Tidak ada data menu.")
        else:
            sorted_data = quick_sort(data[1:])
            table = PrettyTable(["ID_menu", "waktu", "Type", "nama_menu", "Harga"])
            for row in sorted_data:
                table.add_row(row)
            print(table)

# Fungsi untuk mencari data berdasarkan ID dan menampilkannya dalam bentuk tabel
def cari_user(ID):
    found = False
    with open(filename_user, mode='r') as file:
        reader = csv.reader(file)
        table = PrettyTable(["ID", "Nama", "Password", "Type","attempts", "Duit"])
        for row in reader:
            if row[0] == ID:
                table.add_row(row)
                found = True
                break
    if found:
        print("Data User Ditemukan:")
        print(table)
    else:
        print("Data pelanggan tidak ditemukan.")
        
# Fungsi untuk mencari data MENU berdasarkan ID dan menampilkannya dalam bentuk tabel
def cari_menu(ID):
    found = False
    with open(filename_menu, mode='r') as file:
        reader = csv.reader(file)
        table = PrettyTable(["ID_menu", "waktu", "Type", "nama_menu", "Harga"])
        for row in reader:
            if row[0] == ID:
                table.add_row(row)
                found = True
                break
    if found:
        print("Data MENU Ditemukan:")
        print(table)
    else:
        print("Data MENU tidak ditemukan.")

# Fungsi untuk menghapus data USER berdasarkan ID
def hapus_user(id_user):
    data_ditemukan = False
    with open(filename_user, mode='r') as file:
        reader = csv.reader(file)
        data = list(reader)
    with open(filename_user, mode='w', newline='') as file:
        writer = csv.writer(file)
        for row in data:
            if row[0] != id_user:
                writer.writerow(row)
            else:
                data_ditemukan = True
    if data_ditemukan:
        print(f"Data user dengan ID {id_user} berhasil dihapus.")
    else:
        print("Data user tidak ditemukan.")

# Fungsi untuk menghapus data MENU berdasarkan ID
def hapus_menu(id_user):
    data_ditemukan = False
    with open(filename_menu, mode='r') as file:
        reader = csv.reader(file)
        data = list(reader)
    with open(filename_menu, mode='w', newline='') as file:
        writer = csv.writer(file)
        for row in data:
            if row[0] != id_user:
                writer.writerow(row)
            else:
                data_ditemukan = True
    if data_ditemukan:
        print(f"Data MENU dengan ID {id_user} berhasil dihapus.")
    else:
        print("Data MENU tidak ditemukan.")

# Fungsi untuk mengedit data menu
def edit_data_menu(id_user):
    with open(filename_menu, mode='r') as file:
        reader = csv.reader(file)
        data = list(reader)
        user_ditemukan = False
    for i, row in enumerate(data):
        if row[0] == id_user:
            user_ditemukan = True
            print("\nData User Saat Ini:")
            print(f"1. Waktu       : {row[1]}")
            print(f"2. Type  : {row[2]}")
            print(f"3. Nama Menu       : {row[3]}")
            print(f"4. Harga   : {row[4]}")
            # Meminta input untuk perubahan data
            opsi_valid=["siang","malam","pagi",""]
            while True :
                waktu_baru = input("Masukkan waktu Baru (biarkan kosong jika tidak ada perubahan): ").lower() or row[1]   
                if waktu_baru in opsi_valid:
                    break
                else:
                    print("Input tidak valid. Harap masukkan hanya 'siang', 'malam', atau 'pagi'.")
            opsi_validtipe=["1","2",""]
            while True :
                Type_baru = input("Masukkan Type Baru, isi 1 atau 2 (biarkan kosong jika tidak ada perubahan): ") or int(row[2])   
                if Type_baru in opsi_validtipe:
                    break
                else:
                    print("Input tidak valid. Harap masukkan hanya 1 atau 2")
            nama_baru = input("Masukkan Nama Baru (biarkan kosong jika tidak ada perubahan): ") or row[3]
            harga_baru = input("Masukkan nilai Duit Baru (biarkan kosong jika tidak ada perubahan): ") or int(row[4])
            # Memperbarui data
            data[i] = [id_user, waktu_baru,Type_baru,nama_baru, harga_baru]
            break
    if user_ditemukan:
        with open(filename_menu, mode='w', newline='') as file:
            writer = csv.writer(file)
            writer.writerows(data)
        print(f"\nData MENU dengan ID {id_user} berhasil diperbarui.")
    else:
        print("Data MENU tidak ditemukan.")

# Fungsi untuk mengedit data user
def edit_data_user(id_user):
    with open(filename_user, mode='r') as file:
        reader = csv.reader(file)
        data = list(reader)
        user_ditemukan = False
    for i, row in enumerate(data):
        if row[0] == id_user:
            user_ditemukan = True
            print("\nData MENU Saat Ini:")
            print(f"1. Nama       : {row[1]}")
            print(f"2. Password   : {row[2]}")
            print(f"3. Type       : {row[3]}")
            print(f"4. attempts   : {row[4]}")
            print(f"5. Duit       : {row[5]}")
            # Meminta input untuk perubahan data
            nama_baru = input("Masukkan Nama Baru (biarkan kosong jika tidak ada perubahan): ") or row[1]
            password_baru = input("Masukkan Password Baru (biarkan kosong jika tidak ada perubahan): ") or row[2]
            Type_baru = input("Masukkan Type Baru, isi 1 atau 2 (biarkan kosong jika tidak ada perubahan): ") or int(row[3])
            attempts_baru = input("Masukkan attempts Baru (biarkan kosong jika tidak ada perubahan): ") or int(row[4])
            duit_baru = input("Masukkan nilai Duit Baru (biarkan kosong jika tidak ada perubahan): ") or int(row[5])
            # Memperbarui data
            data[i] = [id_user, nama_baru, password_baru, Type_baru,attempts_baru, duit_baru]
            break
    if user_ditemukan:
        with open(filename_user, mode='w', newline='') as file:
            writer = csv.writer(file)
            writer.writerows(data)
        print(f"\nData user dengan ID {id_user} berhasil diperbarui.")
    else:
        print("Data user tidak ditemukan.")

# Main program admin
def admin():
    buat_file_csv()
    #ID= ADMIN
    # password = "ADMIN123":
    while True:
        print("\n============ MENU ADMIN =================")
        print("1. Tambah Data User || 6. Tambah Data Menu")
        print("2. Lihat Data User  || 7. Lihat Data Menu")
        print("3. Cari Data User   || 8. Cari Data Menu")
        print("4. Hapus Data User  || 9. Hapus Data Menu")
        print("5. Edit Data User   || 10. Edit Data Menu")
        print("11. Keluar          ||")
        
        pilihan = input("Masukkan pilihan (1- 11): ")
        
        if pilihan == '1':
            id_user = input("Masukkan ID user: ")
            nama = input("Nama : ")
            password = input("Password : ")
            cek=["1","2"]
            while True:
                Type = input("Masukkan Type 1(Biasa) atau 2 (VIP) (biarkan kosong jika tidak ada perubahan) : ")  
                if Type in cek:
                    break
                else:
                    print("Input tidak valid. Harap masukkan hanya '1', '2'.")
            attempts = input("attempts : ")
            Duit = input("Duit : ")
            tambah_user(id_user, nama,password,Type,attempts,Duit)
        elif pilihan == '2':
            lihat_data_user()
        elif pilihan == '3':
            id_user = input("Masukkan ID User yang ingin dicari: ")
            cari_user(id_user)
        elif pilihan == '4':
            id_user = input("Masukkan ID user yang ingin dihapus: ")
            hapus_user(id_user)
        elif pilihan == '5':
            id_user = input("Masukkan ID user yang ingin diubah datanya: ")
            edit_data_user(id_user)
        elif pilihan == '6':
            id_menu = input("Masukkan ID menu: ")
            waktu = input("Waktu(siang/malam/pagi) : ").lower()
            Type = input("Type 1 atau 2 (biasa/VIP) : ")
            Nama_menu = input("Nama Menu :")
            Harga = input("Harga : ")
            tambah_menu(id_menu, waktu, Type,Nama_menu,Harga)
        elif pilihan == '7':
            lihat_data_menu()
        elif pilihan == '8':
            id_user = input("Masukkan ID MENU yang ingin dicari: ")
            cari_menu(id_user)
        elif pilihan == '9':
            id_user = input("Masukkan ID MENU yang ingin dihapus: ")
            hapus_menu(id_user)
        elif pilihan == '10':
            id_user = input("Masukkan ID MENU yang ingin diubah datanya: ")
            edit_data_menu(id_user)
        elif pilihan == '11':
            clear_screen()
            print("Terima kasih telah menggunakan sistem WARUNG ARDHY.")
            break
        else:
            print("Pilihan tidak valid. Silakan coba lagi.")

# Fungsi untuk menambah dana
def tambah_dana(ID):
    users = load_users()  # Load data pengguna dari file CSV
    try:
        print(f"{users[ID]['Nama']}, Saldo Anda: Rp {users[ID]['Duit']:,}")
        dana = int(input("Masukkan jumlah dana yang ingin ditambahkan: "))
        users[ID]['Duit'] = int(users[ID]['Duit'])
        if dana > 0:
            users[ID]['Duit']+= dana
            save_users(users)
            print(f"Dana berhasil ditambahkan. Saldo Anda sekarang: Rp {users[ID]['Duit']:,}")
        else:
            print("Jumlah dana harus lebih dari 0!")
    except ValueError:
        print("Masukkan angka yang valid!")

def tampilkan_menu(waktu, tipe):
    #Menampilkan menu berdasarkan waktu dan tipe.
    menu_list = []
    table = PrettyTable(["ID_menu", "waktu", "Type", "nama_menu", "Harga"])
    with open(filename_menu, mode='r') as file:
        reader = csv.reader(file)
        next(reader)  # Skip header
        for row in reader:
            if row[1] == waktu and row[2] == tipe:
                table.add_row(row)
                menu_list.append(row)
    if menu_list:
        print("Data MENU Ditemukan:")
        print(table)
    else:
        print("Data MENU tidak ditemukan.")
    return menu_list

def update_saldo_user(username, total_biaya):
    #Mengurangi saldo user di file users.csv dan mengembalikan sisa saldo.
    updated_data = []
    saldo_cukup = False
    sisa_saldo = None
    with open(filename_user, mode='r') as file:
        reader = csv.reader(file)
        header = next(reader)  # Simpan header
        updated_data.append(header)
        for row in reader:
            if row[0] == username:  # Username ada di kolom pertama (ID)
                current_balance = float(row[5])  # Saldo ada di kolom ke 6
                if current_balance >= total_biaya:
                    sisa_saldo = current_balance - total_biaya  # Hitung sisa saldo
                    row[5] = str(sisa_saldo)  # Perbarui saldo
                    saldo_cukup = True
                else:
                    print("Saldo Anda tidak cukup untuk melakukan pembelian.")
            updated_data.append(row)
    
    # Perbarui file users.csv jika saldo cukup
    if saldo_cukup:
        with open(filename_user, mode='w', newline='') as file:
            writer = csv.writer(file)
            writer.writerows(updated_data)
        print("Transaksi berhasil! Saldo telah diperbarui.")
    return saldo_cukup, sisa_saldo

def transaksi_pembelian(menu_list, username):
    #Melakukan transaksi pembelian menu dengan jumlah tertentu.
    if not menu_list:
        print("Tidak ada menu untuk dipilih.")
        return
    
    # Menampilkan ID_menu untuk referensi
    id_list = [menu[0] for menu in menu_list]
    print(f"ID_menu yang tersedia: {', '.join(id_list)}")
    
    # Input ID_menu dan jumlah yang ingin dibeli
    print("Masukkan ID_menu dan jumlah yang ingin dibeli (format: ID=jumlah, pisahkan dengan koma): ")
    pilihan = input("Contoh: 1=2,2=5: ").strip().split(',')
    pilihan_dict = {}
    
    for item in pilihan:
        try:
            id_menu, jumlah = item.split('=')
            pilihan_dict[id_menu.strip()] = int(jumlah.strip())
        except ValueError:
            print(f"Format salah: {item}, akan diabaikan.")
    
    # Proses pembelian
    total_harga = 0
    purchased_items = []
    
    for menu in menu_list:
        id_menu = menu[0]
        if id_menu in pilihan_dict:
            jumlah = pilihan_dict[id_menu]
            harga_satuan = int(menu[4])  # Harga satuan di kolom ke-5
            total_harga += harga_satuan * jumlah
            potongan=0
            if total_harga >= 100000:
                potongan = total_harga * 0.05
                total_harga -=potongan
            purchased_items.append([menu[0], menu[3], harga_satuan, jumlah, harga_satuan * jumlah])
    
    # Tampilkan hasil transaksi
    if purchased_items:
        print("\nDetail Pembelian:")
        table = PrettyTable(["ID_menu", "Nama Menu", "Harga Satuan", "Jumlah", "Total"])
        for item in purchased_items:
            table.add_row(item)
        print(table)
        print(f"Total Harga: Rp{total_harga:,}")
        
        # Kurangi saldo user dan tampilkan sisa saldo
        saldo_cukup, sisa_saldo = update_saldo_user(username, total_harga)
        if saldo_cukup:
            if total_harga + potongan >=100000:
                print(f"***Anda mendapat potongan harga : Rp. {potongan}")
            print(f"Pembelian berhasil! Total biaya: Rp{total_harga:,}")
            print(f"Sisa saldo Anda: Rp{sisa_saldo:,}")
        else:
            print("Transaksi dibatalkan karena saldo tidak cukup.")
    else:
        print("Tidak ada menu yang sesuai dengan ID yang dipilih.")
     
# Fungsi untuk membeli makanan
def pilih_menu(users, username, makanan):
    pilihan = input("Pilih ID menu untuk dibeli (atau 0 untuk batal): ")
    if pilihan.isdigit():
        pilihan = int(pilihan)
        if pilihan == 0:
            print("Pembelian dibatalkan.")
        elif 1 <= pilihan <= len(makanan):
            nama, harga = makanan[pilihan - 1]
            if users[username]["balance"] >= harga:
                users[username]["balance"] -= harga
                save_users(users)
                print(f"Berhasil membeli {nama} seharga Rp {harga:,}. Sisa saldo Anda: Rp {users[username]['balance']:,}")
            else:
                print("Saldo tidak cukup!")
        else:
            print("Pilihan menu tidak valid!")
    else:
        print("Masukkan angka yang valid!")

def pelanggan(waktu_makan,tipe,user_ID):
    waktu_map = {"1": "pagi", "2": "siang", "3": "malam"}
    print("Pilihan waktu:")
    waktu = waktu_map.get(waktu_makan, None)
    if not waktu:
        print("Pilihan waktu tidak valid.")
        return
    # Menampilkan menu berdasarkan waktu dan tipe
    menu_list = tampilkan_menu(waktu, tipe)
    # Melakukan transaksi pembelian
    transaksi_pembelian(menu_list, user_ID)

# Fungsi main menu makanan
def akses_menu(ID, jenis_member):
    users = load_users()  # Load data pengguna dari file CSV
    while True:
        print("\nMenu pelanggan :")
        print("1. Menu Pagi")
        print("2. Menu Siang")
        print("3. Menu Malam")
        print("4. Tambah Dana")
        print("5. Keluar")
        waktu_pilihan = input("Pilih nomor menu (1-5): ")
        waktu_map = {"1": "pagi", "2": "siang", "3": "malam"}
        if waktu_pilihan == "5":
            print("Kembali ke menu utama.")
            break
        elif waktu_pilihan == "4":
            tambah_dana(ID)
           
        elif waktu_pilihan not in waktu_map:
            print("Pilihan tidak valid!")
            break
        else:
            pelanggan(waktu_pilihan,jenis_member,ID)
            
    clear_screen()
# Memulai aplikasi
def main():
    print("=== Program WARUNG ARDHI ===")
    while True:
        print("\n1. Login")
        print("2. Keluar")
        choice = input("Pilih menu: ")
        
        if choice == "1":
            ID, Nama, type = login()
            if type ==1 or type==2:
                member=str(type)
                akses_menu(ID,member)
                
            elif Nama=='ADMIN':
                admin()
            
        elif choice == "2":
            print("Keluar dari program. Sampai jumpa!")
            break
        else:
            print("Pilihan tidak valid!xxx")

if __name__ == "__main__":
    main()
