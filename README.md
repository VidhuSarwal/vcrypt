# Vcrypt

A cloud storage aggregator that splits, obfuscates, and distributes files across multiple Google Drive accounts. No single provider ever holds a complete, readable file.

---

## How It Works

1. **Obfuscate** — A 32-byte seed initializes a ChaCha20-DRBG that injects deterministic noise into the file at calculated offsets.
2. **Split** — The obfuscated file is broken into fragments using your chosen strategy (Greedy, Balanced, Proportional, or Manual).
3. **Distribute** — Fragments are uploaded in parallel across your linked Google Drive accounts.
4. **Key File** — A `.2xpfm.key` file is generated containing the seed and chunk map. This never touches the server — you are solely responsible for it.

To reconstruct a file, you provide the Key File and Vcrypt fetches and de-obfuscates the fragments.

---

## Tech Stack

| Layer | Technology |
| :--- | :--- |
| Backend | Go 1.24, MongoDB, JWT |
| Frontend | React 18, TypeScript, Vite |
| Styling | Tailwind CSS, shadcn/ui |
| Cryptography | ChaCha20-DRBG, AES-256-GCM, Bcrypt |
| Cloud | Google Drive API v3 |

---

## Project Structure

```
vcrypt/
├── vcrypt_backend/
│   ├── cmd/server/           # Entry point
│   ├── internal/
│   │   ├── auth/             # JWT, login, signup
│   │   ├── fileprocessor/    # Obfuscation, chunking, key file generation
│   │   ├── oauth/            # Google OAuth2 flow
│   │   └── store/            # MongoDB repository layer
│   └── test_routes.sh        # curl-based API test suite
└── vcrypt_frontend/
    └── src/                  # React app (Dashboard, Upload Wizard, Reconstruct)
```

---

## Setup

### Prerequisites
- Go 1.24+
- Node.js 18+
- MongoDB (local or Atlas)
- Google Cloud project with Drive API and OAuth credentials

### Backend
```bash
cd vcrypt_backend
cp .env.example .env        # Fill in MONGO_URI, JWT_SECRET, GOOGLE_CLIENT_ID, GOOGLE_CLIENT_SECRET
go mod tidy
go run cmd/server/main.go   # Runs on :5555
```

### Frontend
```bash
cd vcrypt_frontend
npm install
npm run dev                 # Runs on :5173
```

---

## Key Concepts

- **Key File (`.2xpfm.key`)** — JSON file holding the obfuscation seed and chunk-to-drive mapping. Never stored server-side. Losing it means losing access to your data.
- **Obfuscation Seed** — 256-bit value used to reproduce the exact noise pattern for reconstruction.
- **Fragment (`.2xpfm`)** — A piece of the obfuscated file stored on Google Drive. Meaningless without the Key File.

---

## Known Limitations

- The full file is temporarily written to the server's disk before splitting, capping max file size to available server storage.
- Very large files may cause the `/finalize` request to time out during processing.

---

## Notes for Developers

- Changing `OBFUSCATION_BLOCK_SIZE` or the algorithm in `internal/fileprocessor` will break compatibility with existing Key Files.
- Google OAuth scope required: `https://www.googleapis.com/auth/drive.file`
- To add a new storage provider (OneDrive, S3, etc.), implement a new manager in `internal/drivemanager` and update the `ChunkPlan` logic.