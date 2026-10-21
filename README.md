# CRYSTALFORGE Tower Defense - Main Menu

Bagian **Main Menu / Landing Page** untuk project game tower defense (mata kuliah Game Cerdas).

## Isi menu

- **Play**: pindah ke scene gameplay
- **Options**:
  - Slider Music dan SFX
  - Toggle Fullscreen
  - Pilihan bahasa (English / Bahasa Indonesia)
- **Quit**: keluar dari game

Semua pengaturan (volume, fullscreen, bahasa) tersimpan otomatis lewat `PlayerPrefs`.

## Versi yang dipakai

- Unity: **[isi versi Unity, contoh 2022.3.x LTS]**
- Package: TextMeshPro (wajib), Unity UI (bawaan)

> Pakai versi Unity yang sama dengan project utama supaya tidak ada error kompatibilitas.

---

## Struktur folder

```
Assets/
└── MainMenu/
    ├── Scenes/
    │   └── MainMenu.unity
    ├── Scripts/
    │   ├── MainMenu.cs
    │   ├── OptionsMenu.cs
    │   ├── LocalizationManager.cs
    │   ├── LocalizedText.cs
    │   └── AudioSettingsLoader.cs   (untuk scene gameplay, lihat bawah)
    ├── Audio/
    │   └── MainMixer.mixer
    └── Art/
        └── (background, font, dll)
```

> Sesuaikan dengan struktur folder yang sebenarnya di repo ini.

---

## Cara memindahkan ke project utama

Pilih **salah satu** cara.

### Cara A (disarankan): Unity Package

1. Buka project ini di Unity.
2. Di Project window, klik kanan folder `Assets/MainMenu` lalu **Export Package...**
3. Pastikan **Include dependencies** dicentang, lalu klik **Export**.
4. Di project utama: **Assets > Import Package > Custom Package...**, pilih file `.unitypackage` tadi, lalu **Import**.

Cara ini menjaga semua referensi (script, mixer, gambar) tetap tersambung.

### Cara B: Copy folder manual

1. Tutup Unity di project tujuan.
2. Copy seluruh folder `Assets/MainMenu` **beserta file `.meta`-nya** (termasuk `MainMenu.meta`) ke folder `Assets/` project utama.
3. Buka Unity dan tunggu proses import selesai.

> **Jangan** copy lewat Unity tanpa `.meta`, atau referensi bisa putus (muncul "Missing" di Inspector).

---

## Setelah dipindah: checklist wajib

- [ ] **TextMeshPro sudah di-import.** Kalau muncul popup *TMP Importer*, klik **Import TMP Essentials**.
- [ ] **Build Settings** (File > Build Settings), urutan scene:
  - `0` = `MainMenu`
  - `1` = scene gameplay (nama harus sesuai, lihat di bawah)
- [ ] **Nama scene gameplay** sama dengan yang dipakai tombol Play. Buka scene `MainMenu`, pilih objek `MenuManager`, lalu isi **Game Scene Name** di komponen `MainMenu` dengan nama scene gameplay yang asli.
- [ ] Di scene `MainMenu`, objek ini **harus ada**: `LocalizationManager`, `MenuManager`, dan `EventSystem`.
- [ ] Referensi `MenuManager` terisi semua (tidak ada yang `None`): Main Panel, Options Panel, Audio Mixer, Music Slider, SFX Slider, Fullscreen Toggle, Language Dropdown.
- [ ] Tidak ada error merah di Console.

---

## Audio: supaya volume dari Options berlaku di dalam game

Volume diatur lewat **AudioMixer `MainMixer`** dengan dua group dan dua parameter:

| Group | Parameter (exposed) | Dipakai untuk |
|---|---|---|
| Music | `MusicVolume` | Musik latar |
| SFX | `SFXVolume` | Efek suara (tembakan, klik, dll) |

Yang harus dilakukan di scene gameplay:

1. **Arahkan Output AudioSource** ke group yang sesuai (musik ke `Music`, efek suara ke `SFX`). Kalau Output tidak diarahkan, slider tidak berpengaruh ke suara itu.
2. Buat Empty GameObject `AudioSettings` di scene gameplay, lalu pasang script `AudioSettingsLoader` dan isi field **Audio Mixer** dengan `MainMixer`.

```csharp
using UnityEngine;
using UnityEngine.Audio;

public class AudioSettingsLoader : MonoBehaviour
{
    [SerializeField] private AudioMixer audioMixer;

    private void Start()
    {
        ApplyVolume("MusicVolume");
        ApplyVolume("SFXVolume");
    }

    private void ApplyVolume(string parameterName)
    {
        // Ambil nilai yang disimpan dari menu Options (default 0.75)
        float value = PlayerPrefs.GetFloat(parameterName, 0.75f);
        audioMixer.SetFloat(parameterName, Mathf.Log10(value) * 20f);
    }
}
```

Script ini perlu supaya volume tetap benar kalau scene gameplay dijalankan langsung dari Editor.

---

## Bahasa (opsional untuk scene gameplay)

Kalau teks di gameplay (misalnya HUD, nama tower, pesan Game Over) ingin ikut berganti bahasa:

1. Tambahkan entri baru di dictionary pada `LocalizationManager.cs`. Format: `{ "key", new[] { "Teks English", "Teks Indonesia" } }`.
2. Pasang script `LocalizedText` di objek teksnya, lalu isi **Key**-nya.

Catatan:
- `LocalizedText` dipasang di objek yang punya komponen **TextMeshPro - Text (UI)**. Untuk tombol, pasang di child `Text (TMP)`, bukan di Button.
- `LocalizationManager` memakai `DontDestroyOnLoad`, jadi bahasa terbawa dari menu ke gameplay. Kalau scene gameplay dijalankan langsung dari Editor, objek `LocalizationManager` harus ada di scene itu juga (duplikat otomatis dihapus saat pindah dari menu).

---

## Kembali ke Main Menu dari gameplay

Untuk tombol Pause atau Game Over, tinggal panggil:

```csharp
using UnityEngine.SceneManagement;

SceneManager.LoadScene("MainMenu");
```

---

## Key PlayerPrefs yang dipakai

| Key | Tipe | Isi |
|---|---|---|
| `MusicVolume` | float (0.0001 sampai 1) | Volume musik |
| `SFXVolume` | float (0.0001 sampai 1) | Volume efek suara |
| `Fullscreen` | int (0 atau 1) | Mode layar penuh |
| `Language` | int (0 = English, 1 = Indonesia) | Bahasa terpilih |

---

## Troubleshooting

| Masalah | Solusi |
|---|---|
| `Scene 'xxx' couldn't be loaded` | Scene belum masuk Build Settings, atau nama di **Game Scene Name** salah. |
| `NullReferenceException` di `LocalizedText` | Objek `LocalizationManager` tidak ada di scene. |
| Volume tidak berubah | Nama parameter mixer tidak persis `MusicVolume` / `SFXVolume`, atau Output AudioSource belum diarahkan ke group mixer. |
| Script atau gambar "Missing" setelah import | File `.meta` tidak ikut ter-copy. Pakai Cara A (Unity Package). |
| Teks UI kotak-kotak atau tidak muncul | TMP Essentials belum di-import. |
| Tombol tidak bisa diklik | `EventSystem` hilang dari scene. |

---

## Kontak

Ada yang error atau bingung? Hubungi **[nama kamu / kontak]**.
