# Claude CLI

Claude langsung di terminal (Linux / Ubuntu / Termux) memakai **Anthropic API resmi** lewat SDK Python resmi.
UI berwarna (Rich), respons streaming, riwayat prompt dengan panah ↑/↓ (prompt_toolkit), memori percakapan di SQLite,
ringkasan konteks otomatis, sistem command `/`, dan export Markdown/JSON/TXT.

## Instalasi

```bash
git clone <url-repo-anda> claude-cli
cd claude-cli

python3 -m venv .venv
source .venv/bin/activate

pip install -r requirements.txt

cp .env.example .env

nano .env          # isi ANTHROPIC_API_KEY=sk-ant-...

python claude.py
```

Butuh Python 3.10+. Contoh Termux: `pkg install python` lalu langkah di atas. Paket `anthropic` memakai
dependensi native (pydantic-core, jiter); jika `pip` di Termux meminta compiler, jalankan
`pkg install rust clang make libffi openssl` dulu (proses build bisa beberapa menit).

Opsi baris perintah: `--new` (sesi baru), `--continue` (lanjut sesi terakhir), `--session ID`, `--debug`, `--version`.

## Konfigurasi (`.env`)

| Variabel | Default | Keterangan |
|---|---|---|
| `ANTHROPIC_API_KEY` | – | API key Anda |
| `CLAUDE_MODEL` | `claude-sonnet-4-5` | Model apa pun yang tersedia untuk akun Anda |
| `MAX_TOKENS` | `4096` | Batas token jawaban |
| `DATABASE_PATH` | `claude.db` | Lokasi SQLite (relatif terhadap folder proyek) |
| `DEBUG` | `false` | `true` = tampilkan traceback + log detail |
| `ANTHROPIC_API_KEY_1..N` | – | Credential tambahan milik akun/organisasi Anda sendiri |
| `ALLOW_CREDENTIAL_FALLBACK` | `false` | Lihat bagian Credential |
| `REQUEST_TIMEOUT` | `120` | Detik |
| `CONTEXT_TOKEN_BUDGET` | `12000` | Di atas estimasi ini, pesan lama diringkas |
| `RECENT_MESSAGES` | `8` | Jumlah pesan terbaru yang selalu dikirim utuh |
| `LOG_DIR` / `EXPORT_DIR` | `logs` / `exports` | |
| `SYSTEM_PROMPT` | (bawaan) | Ganti system prompt |

### Credential
Beberapa key boleh diisi (`ANTHROPIC_API_KEY_1`, `_2`, …). Perpindahan ke key berikutnya **hanya** terjadi jika
`ALLOW_CREDENTIAL_FALLBACK=true` **dan** key aktif ditolak sebagai tidak valid (HTTP 401). Alasannya ditampilkan.
Rate limit, kuota, atau masalah billing **tidak pernah** memicu perpindahan key.
API key tidak pernah dicetak, disimpan di SQLite, ditulis ke log (dimask), maupun ke export.

## Command

| Command | Fungsi |
|---|---|
| `/help` | Daftar command dan tombol |
| `/clear` | Bersihkan terminal (escape sequence), tampilkan header lagi |
| `/reset` | Hapus semua pesan + ringkasan sesi ini (dengan konfirmasi) |
| `/new` | Mulai sesi baru |
| `/history [N\|all]` | Riwayat pesan sesi ini |
| `/sessions` | Daftar sesi tersimpan |
| `/memory [on\|off\|summary\|summarize\|clear]` | Info & kontrol memori/ringkasan |
| `/model [nama\|list\|reset]` | Lihat/ganti model (divalidasi ke API), `list` = daftar model dari API |
| `/status` | Cek koneksi API (request tanpa token), model, sesi, database |
| `/save [judul]` | Beri judul sesi (semua pesan sudah otomatis tersimpan) |
| `/load [id\|nomor]` | Buka sesi lain (tanpa argumen: menu panah ↑/↓) |
| `/export [md\|json\|txt\|all]` | Simpan ke `exports/session_YYYYMMDD_HHMM.*` |
| `/session` | Info sesi aktif |
| `/quit`, `/exit` | Keluar |

Teks yang diawali `/` tetapi bukan nama command (mis. `/usr/bin/ls`) dikirim ke Claude sebagai pesan biasa.
Untuk mengirim pesan yang diawali nama command, beri spasi di depan.

## Keyboard

`Enter` kirim · `Ctrl+J` / `Alt+Enter` / `\` lalu `Enter` = baris baru · `↑`/`↓` riwayat prompt (disimpan di
`.claude_history`) · `Tab` autocomplete command · `Ctrl+C` hentikan jawaban yang sedang mengalir (jawaban parsial tetap
disimpan) / bersihkan input · `Ctrl+D` keluar.

Saat menjalankan ulang dan sudah ada sesi: menu **Continue last / Start new / Select session** (↑/↓, Enter, atau angka 1-9).

## Cara kerja memori & konteks

Semua pesan disimpan di SQLite (`sessions`, `messages`, `summaries`, `settings`). Yang dikirim ke Claude bukan seluruh
database: jika percakapan pendek, dikirim system prompt + seluruh percakapan; jika estimasi token melewati
`CONTEXT_TOKEN_BUDGET`, pesan lama diringkas oleh Claude (tujuan, keputusan, konteks teknis, file/proyek, instruksi,
hasil, hal yang belum selesai), ringkasan disimpan, lalu dikirim **ringkasan + pesan terbaru** apa adanya.
`/memory off` mengirim tiap prompt tanpa riwayat (pesan tetap disimpan lokal).

## Error & log

Error API (autentikasi, rate limit, timeout, koneksi, server, model tidak valid, respons kosong/rusak) tampil sebagai
panel singkat. Detail teknis ada di `logs/claude-cli.log` (key dimask). `DEBUG=true` atau `--debug` menampilkan traceback.

## Test

```bash
python -m unittest discover -s tests -v      # atau: pytest
```
Test tidak memanggil API sungguhan.

## Struktur

```
claude.py  config.py  client.py  database.py  memory.py  session.py
commands.py  ui.py  errors.py  requirements.txt  .env.example  .gitignore
tests/  logs/  exports/
```
# multi-api-key-claude
