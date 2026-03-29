# FAIt_Training
Repository untuk preprocessing, training dan konversi model untuk aplikasi FAIt

Tutorial Pipeline Pelatihan YOLO:
Ikuti langkah-langkah di bawah ini untuk menjalankan pipeline pelatihan model deteksi objek menggunakan YOLO dan integrasi Weights & Biases (WandB).

1. Persiapan Akun dan API Key
Weights & Biases (WandB): Siapkan akun di wandb.ai. Ambil API Key dari pengaturan profil.
Jika menggunakan Google Colab, masukkan API Key tersebut ke dalam menu Secrets (ikon kunci di panel kiri) dengan nama WANDB_API_KEY dan aktifkan akses notebook.

2. Instalasi Dependensi
Jalankan perintah berikut untuk menginstal library yang diperlukan seperti ultralytics untuk YOLO, wandb untuk tracking, dan ray untuk optimasi model:
    pip install ultralytics "ray[tune]" wandb sng4onnx onnx_graphsurgeon ai-edge-litert onnxruntime-gpu

3. Konfigurasi Environment
Aktifkan integrasi WandB pada pengaturan YOLO agar semua log pelatihan terkirim secara otomatis ke dashboard Anda:
    !yolo settings wandb=True

4. Manajemen Dataset dengan WandB
Gunakan kelas WandbHandler yang tersedia di notebook untuk mengelola dataset secara otomatis:
Download Dataset: Mengunduh dataset langsung dari artifact WandB ke direktori lokal /content/dataset/.
Upload/Update: Mengunggah versi terbaru dataset atau file spesifik ke WandB untuk kontrol versi data yang lebih baik.

5. Menjalankan Training
Inisialisasi model YOLO (misalnya yolo26n.pt) dan mulai proses pelatihan. Pipeline ini sudah dilengkapi dengan fungsi kustom untuk mencatat metrik tambahan seperti Precision-Recall Curve ke dalam tabel WandB agar hasil evaluasi lebih detail.

6. Monitoring dan Validasi

Dashboard WandB: Pantau grafik loss dan akurasi secara real-time melalui link project WandB yang muncul saat running.
Cek Hasil: Setelah pelatihan selesai, hasil validasi dan bobot model terbaik akan tersimpan di folder runs/detect/val.

Catatan: Pastikan struktur folder dataset Anda sudah sesuai dengan format yang diminta oleh YOLO (memiliki file .yaml konfigurasi) sebelum memulai pelatihan.