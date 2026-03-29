# FAIt_Training
Repository untuk preprocessing, training dan konversi model untuk aplikasi FAIt

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