# FAIt_Training
Repository untuk preprocessing, training dan konversi model untuk aplikasi FAIt

Tutorial Penggunaan Pipeline Preprocessing

1. Menyiapkan Akun Weights & Biases (W&B)
   Pastikan Anda telah memiliki akun pada platform Weights & Biases (W&B) serta API Key yang aktif.

2. Konfigurasi API Key di Google Colab
   Masukkan *API Key* W&B ke dalam menu *Secrets* di Google Colab dengan mengisi *name* dan *value* sesuai dengan kredensial yang dimiliki.

3. Persiapan Dataset
   Siapkan dataset dengan format YOLOv8/YOLO format (yolo26) sesuai dengan kebutuhan pipeline.

4. Menjalankan Dependensi dan Fungsi
   Eksekusi seluruh blok kode yang berisi instalasi dependensi serta definisi fungsi yang diperlukan pada notebook Google Colab.

5. Mengunduh Dataset
   Lakukan pengunduhan dataset dari sumber yang tersedia, seperti W&B, Google Drive, atau Roboflow, dengan memastikan format dataset tetap konsisten (yolo26).

6. Transformasi Dataset ke DataFrame
   Jalankan fungsi yang tersedia untuk menggabungkan dan mengonversi dataset menjadi bentuk *DataFrame* guna memudahkan proses analisis.

7. Eksplorasi dan Analisis Data (EDA)
   Lakukan proses eksplorasi data (*Exploratory Data Analysis*) serta analisis dan manipulasi data sesuai kebutuhan.

8. Web Scraping Data Nilai Gizi Berdasarkan Label
   Jalankan kode *web scraping* untuk mengambil informasi nilai gizi berdasarkan label makanan dari situs FatSecret Indonesia. Data yang diambil dapat meliputi kalori, protein, lemak, dan karbohidrat.

9. Konversi Kembali ke Format Dataset
   Setelah seluruh proses analisis dan penambahan data selesai, jalankan fungsi untuk mengonversi kembali *DataFrame* menjadi format dataset semula.

10. Upload Dataset ke W&B
    Terakhir, jalankan fungsi yang tersedia untuk mengunggah dataset hasil preprocessing ke platform W&B.


Tutorial Implementasi Pipeline YOLO & Weights & Biases (WandB)

Tutorial ini akan dari tahap persiapan lingkungan hingga evaluasi model menggunakan integrasi pelacakan eksperimen otomatis.

1. Persiapan Kredensial (WandB API Key)
Sebelum mulai menyentuh kode, kita memerlukan akses ke Weights & Biases untuk mencatat log training.
- Dapatkan API Key: Masuk ke wandb.ai/settings dan salin API Key.
- Konfigurasi di Google Colab:
    1. Klik ikon kunci (Secrets) di panel sebelah kiri.
    2. Tambahkan new secret dengan nama WANDB_API_KEY.
    3. Masukkan API Key di kolom Value.
    4. Aktifkan centang Notebook access agar script dapat membaca kunci tersebut secara otomatis.
- Konfigurasi di Kaggle:
    1. Buka menu Add-ons > Secrets.
    2. Tambahkan label WANDB_API_KEY dengan nilai API Key.

2. Instalasi dan Setup Environment
Langkah pertama dalam notebook adalah menginstal semua dependensi yang diperlukan. Pipeline ini membutuhkan ultralytics untuk YOLO machine dan beberapa library tambahan untuk optimasi dan penanganan format model.

    #Instalasi library utama
    !pip install ultralytics "ray[tune]" wandb sng4onnx onnx_graphsurgeon ai-edge-litert onnxruntime-gpu

Setelah instalasi, aktifkan integrasi WandB di pengaturan YOLO agar setiap metrik pelatihan (seperti loss dan mAP) langsung terkirim ke dashboard.

    #Mengaktifkan tracking WandB pada YOLO
    !yolo settings wandb=True

3. Manajemen Dataset dengan WandbHandler

Notebook ini menggunakan kelas khusus bernama WandbHandler untuk menyederhanakan interaksi dengan dataset yang disimpan sebagai artifact di WandB. Ini memastikan seluruh tim menggunakan versi data yang sama.

- Inisialisasi: Buat objek handler dengan menentukan nama proyek dan nama run.
- Download Dataset: Gunakan fungsi download_from_wandb('nama_dataset'). Skrip akan memeriksa apakah dataset sudah ada di direktori /content/dataset/ untuk menghindari pengunduhan ulang yang sia-sia.
- Update/Upload: Jika Anda melakukan perubahan pada data lokal, gunakan upload_to_wandb atau update_file untuk menyinkronkan perubahan tersebut kembali ke awan.

4. Proses Pelatihan (Training)

Training model dimulai dengan memanggil fungsi YOLO. Pipeline ini mendukung fitur resume jika training terhenti di tengah jalan.

    from ultralytics import YOLO

    # Memuat model (misal: versi nano untuk kecepatan)
    model = YOLO('yolo11n.pt')

    # Memulai pelatihan
    results = model.train(
        data='/content/dataset/data.yaml', 
        epochs=100, 
        imgsz=640,
        project='nama_proyek_anda',
        name='eksperimen_1'
    )

5. Visualisasi Metrik Custom

Salah satu keunggulan pipeline ini adalah adanya fungsi _custom_table dan _plot_curve. Fungsi ini akan membuat tabel visualisasi kustom di WandB, seperti Precision-Recall Curve, yang jauh lebih detail daripada grafik standar YOLO.
Metrik ini akan membantu menganalisis kelas mana yang memiliki performa paling lemah sehingga dapat mengetahui bagian mana dari dataset yang perlu diperbaiki.

6. Evaluasi dan Penyimpanan Model

Setelah pelatihan selesai:
    1. Cek Dashboard: Buka link proyek di WandB untuk melihat perbandingan antar eksperimen.
    2. Validasi: Jalankan validasi pada set pengujian menggunakan bobot terbaik (best.pt) yang tersimpan di direktori runs/detect/.
    3. Ekspor: Jika diperlukan, model dapat diekspor ke format ONNX atau TFLite untuk penggunaan di perangkat mobile atau web menggunakan pustaka yang sudah diinstal di awal.