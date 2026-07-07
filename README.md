# BackupSheep

BackupSheep is a self-hosted backup automation platform for databases, websites, servers, cloud snapshots, and offsite storage destinations. It provides a Django web console, scheduled Celery jobs, provider integrations, retention rules, and restore-oriented backup history.

## What it backs up

- Databases: PostgreSQL, MySQL, and MariaDB dumps.
- Websites and servers: FTP, FTPS, SFTP, and SSH based file backups.
- Cloud resources: snapshots for providers such as AWS, DigitalOcean, Hetzner, Linode, Vultr, UpCloud, Oracle Cloud, Google Cloud, and OVH.
- SaaS and app data: WordPress and Basecamp support.
- Storage destinations: S3-compatible storage, Backblaze B2, Wasabi, Cloudflare R2, Google Cloud Storage, Azure Blob, Google Drive, Dropbox, OneDrive, pCloud, IBM COS, Oracle, Scaleway, and more.

See `docs/providers.md` for the provider matrix.

## Architecture

The Docker Compose stack runs one application image as multiple service roles:

| Service | Purpose |
| --- | --- |
| `app` | Django web console served by Gunicorn and WhiteNoise. |
| `migrate` | One-shot database migration service. |
| `worker-cloud` | Provider API snapshot tasks and default queue work. |
| `worker-database` | Database dump jobs. |
| `worker-files` | Website and file backup jobs. |
| `worker-storage` | Uploads completed backup artifacts to storage destinations. |
| `worker-logs` | Log persistence, notifications, and pruning. |
| `beat` | Celery scheduler. Keep exactly one instance. |
| `db` | PostgreSQL database. |
| `redis` | Celery broker. |

## Stack

| Area | Technology |
| --- | --- |
| Backend | Django 6, Django REST Framework |
| Jobs | Celery, Redis, django-celery-beat |
| Database | PostgreSQL |
| Web server | Gunicorn, WhiteNoise |
| UI assets | Tailwind CSS |
| Provider SDKs | boto3, Google Cloud, Azure, Dropbox, Oracle, OVH, SSH/FTP libraries |
| Container runtime | Docker Compose |

## Quick start

Prerequisites:

- Docker with the Compose plugin
- Git

```bash
cp .env_sample .env
```

Edit `.env` and set at least:

```text
DJANGO_SECRET_KEY=<long random secret>
DB_PASSWORD=<database password>
```

Then start the stack:

```bash
docker compose up --build
```

Open:

```text
http://localhost:8000
```

The first-run flow guides you through creating the admin account and adding backup sources and storage destinations.

## Configuration

Start with `.env_sample`. The Compose file expects service names such as `db` and `redis` for database and broker connectivity.

Important production settings include:

- `DJANGO_SECRET_KEY`
- `ALLOWED_HOSTS`
- `DJANGO_HTTPS`
- Database credentials
- Email settings
- Storage provider credentials
- Celery broker/result settings

See:

- `docs/configuration.md`
- `docs/deployment.md`
- `docs/first-run.md`

## Operations

Scale upload throughput by increasing storage workers:

```bash
docker compose up -d --scale worker-storage=4
```

Keep `beat` as a singleton. Running multiple beat schedulers can dispatch scheduled backups more than once.

Backup work files are shared through the `backup_workdir` volume so dump workers and upload workers can coordinate on the same host.

## Development

Install Python dependencies from `requirements.txt` and frontend dependencies from `package.json` when working outside Docker.

```bash
pip install -r requirements.txt
npm install
npm run build:css
python manage.py test
```

## Documentation

- `docs/installation.md` - install guide
- `docs/configuration.md` - environment variables
- `docs/first-run.md` - setup wizard
- `docs/usage.md` - creating sources, destinations, schedules, and restores
- `docs/providers.md` - supported providers
- `docs/deployment.md` - production deployment notes
- `docs/scaling.md` - worker queues and scaling
- `docs/troubleshooting.md` - common issues

## License

BackupSheep is licensed under the GNU General Public License v3.0. See `LICENSE` for the full terms.
