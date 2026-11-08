# SCOPOS Studio — self-hosted Cap

Fork of [CapSoftware/Cap](https://github.com/CapSoftware/Cap) (open-source Loom alternative). `main` tracks upstream; everything SCOPOS-specific lives in this `scopos/` folder. City building: **The Recording Studio** (Main Street, `/studio/`).

## Where it runs
- Oracle VM `132.145.150.53` (private NIC `10.0.0.121`), folder `/opt/data/projects/scopos-studio/scopos`.
- `docker compose` stack: `cap-web` (Next.js, 127.0.0.1:3000), `media-server`, `mysql` (8.0, volume `mysql-data`), `minio` (S3, volume `minio-data`, 127.0.0.1:9000/9001), `caddy` (TLS, profile `public`).
- Secrets: `scopos/.env` (gitignored, chmod 600). Never commit it.

## Addresses
| Host | Goes to |
| --- | --- |
| `studio.scoposhg.com` | Cap web app (share pages, dashboard, desktop app server URL) |
| `studio-files.scoposhg.com` | MinIO S3 API, used by presigned video URLs (bucket stays private) |

Go-live checklist:
1. DNS (cPanel): `A studio → 132.145.150.53`, `A studio-files → 132.145.150.53`.
2. OCI security list: ingress TCP 80 + 443 from 0.0.0.0/0.
3. `sudo docker compose --profile public up -d caddy` (Caddy gets Let's Encrypt certs on first request).
4. Email sign-in: set `RESEND_API_KEY` + `RESEND_FROM_DOMAIN` (or Google OAuth `GOOGLE_CLIENT_ID/SECRET`) in `.env`, then `sudo docker compose up -d cap-web`. Until then login links are printed in `sudo docker compose logs cap-web`.

## Ops
```bash
cd /opt/data/projects/scopos-studio/scopos
sudo docker compose ps
sudo docker compose logs -f cap-web
sudo docker compose pull && sudo docker compose up -d     # upgrade (pin CAP_WEB_TAG in .env for stability)
```
MinIO console: `ssh -L 9001:127.0.0.1:9001 oracle`, then http://localhost:9001 (user/secret = `S3_ACCESS_KEY`/`S3_SECRET_KEY`).

## Storage
Cap needs MySQL (its Drizzle schema) + S3-compatible object storage. Convex is not a supported backend. To move video files off the VM later (e.g. Cloudflare R2, zero egress fees for outside viewers), point `S3_*` in `.env` at the new bucket and copy the objects with `mc mirror`.

## License
Cap is AGPLv3. Running it unmodified is fine. If we change Cap's code and serve it to outside viewers, the modified source must be offered to them: keep changes in this public fork.
