# Local-Server — shared local services

Centralized local services สำหรับ **ทุกโปรเจค** บนเครื่องนี้

## กฎเหล็ก

1. **ทุกโปรเจคใช้ service ชุดนี้ร่วมกัน** — ห้ามสร้าง container ของ postgres / mysql / mongodb / redis เองในโปรเจคเด็ดขาด
2. แต่ละโปรเจคแยกข้อมูลด้วย **database ของตัวเอง** (เช่น `myapp_dev`) ไม่ใช่ instance ของตัวเอง
3. มี service ใหม่ที่ต้องใช้ร่วมกัน → **มาเพิ่มที่ `docker-compose.yml` ไฟล์นี้** ที่เดียว

## วิธีใช้

```bash
docker compose up -d        # start ทั้งหมด (background)
docker compose ps           # ดูสถานะ + health
docker compose logs -f      # ดู log
docker compose stop         # หยุด (ข้อมูลยังอยู่)
docker compose down         # ลบ container (volume/ข้อมูลยังอยู่)
docker compose down -v      # ⚠️ ลบข้อมูลทั้งหมดด้วย
```

## Services & connection strings

ค่า credential default อยู่ใน [.env](.env) (แก้ได้)

| Service    | Host        | Port  | User / Pass         |
|------------|-------------|-------|---------------------|
| PostgreSQL | `localhost` | 5432  | `postgres` / `postgres` |
| MySQL      | `localhost` | 3306  | `app` / `app` (root: `root`) |
| MongoDB    | `localhost` | 27017 | `root` / `root`     |
| Redis      | `localhost` | 6379  | (ไม่มี password)     |
| MinIO (S3) | `localhost` | 9000  | `minioadmin` / `minioadmin` (console: 9001) |
| 9router    | `localhost` | 20128 | login: password `123456` (เปลี่ยนใน `.env`) |

```
postgresql://postgres:postgres@localhost:5432/<db>
mysql://app:app@localhost:3306/<db>
mongodb://root:root@localhost:27017/<db>?authSource=admin
redis://localhost:6379
```

MinIO — S3 endpoint `http://localhost:9000`, web console `http://localhost:9001`. แต่ละโปรเจคใช้ **bucket ของตัวเอง** (`region` อะไรก็ได้, ต้องใช้ path-style):

```
S3_ENDPOINT=http://localhost:9000
S3_ACCESS_KEY_ID=minioadmin
S3_SECRET_ACCESS_KEY=minioadmin
S3_FORCE_PATH_STYLE=true
```

9router — local AI router/proxy ([decolua/9router](https://github.com/decolua/9router)) ต่อ coding tools (Claude Code / Cursor / Cline) เข้าหลาย AI provider พร้อม smart fallback. Dashboard + API ที่ `http://localhost:20128` (login ครั้งแรก password `123456`). ตั้ง base URL ของเครื่องมือมาที่ endpoint นี้.

## สร้าง database ต่อโปรเจค

```bash
# Postgres
docker exec -it local-postgres createdb -U postgres myapp_dev

# MySQL
docker exec -it local-mysql mysql -uroot -proot -e "CREATE DATABASE myapp_dev;"

# MongoDB — สร้าง db อัตโนมัติเมื่อเขียนครั้งแรก ไม่ต้อง pre-create
```

## เชื่อมจากโปรเจคอื่น (container-to-container)

ถ้าโปรเจคอื่นรันใน Docker ด้วย ให้ join network `local-server` แล้วเรียกผ่านชื่อ service (`postgres`, `mysql`, `mongodb`, `redis`) แทน `localhost`:

```yaml
# docker-compose.yml ของโปรเจคอื่น
services:
  app:
    networks: [local-server]
networks:
  local-server:
    external: true
```
