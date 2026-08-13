# Docker — Complete Interview Prep Guide (Hinglish)

---

## 1. Docker Hai Kya? (Simple Words Mein)

Socho tumne apne laptop pe ek FastAPI project banaya. Usme Python 3.11, specific libraries, specific OS settings hain. Ab jab tum yeh project kisi doosre server pe deploy karoge ya friend ko doge, uske system pe Python version different ho sakta hai, library version mismatch ho sakta hai — aur bolta hai **"works on my machine but not on yours"**.

**Docker is problem ko solve karta hai.** Docker tumhare poore application ko — code, dependencies, OS libraries, settings — ek **box (container)** mein pack kar deta hai. Yeh box jahan bhi jaaye (tumhare laptop, server, cloud), **exactly same tarike se chalega**.

> **Interview mein bologe:** "Docker ek containerization platform hai jo application aur uski dependencies ko ek isolated, portable unit (container) mein package karta hai, taaki wo kisi bhi environment mein consistently chale."

---

## 2. Docker vs Virtual Machine (Sabse Common Interview Question)

Yeh interview mein **90% chances hai poochha jaayega**. Concept clear rakho.

| Point | Virtual Machine (VM) | Docker Container |
|---|---|---|
| OS | Har VM ka apna **poora OS** hota hai (Guest OS) | Host OS ka kernel **share** karta hai |
| Size | Heavy (GBs) | Lightweight (MBs) |
| Boot time | Minutes | Seconds |
| Isolation | Hardware-level (via Hypervisor) | Process-level (via OS kernel features) |
| Resource usage | Zyada (har VM ko apna RAM/CPU chahiye) | Kam (sab containers host kernel share karte hain) |

**Why important (interviewer ye "why" bhi poochhega):**
VM chalta hai **Hypervisor** ke upar, jo hardware ko virtualize karta hai — isliye har VM ko apna poora OS chahiye. Docker container chalta hai **Docker Engine** ke upar, jo host machine ke OS kernel ko use karta hai but process ko isolate kar deta hai using Linux features (namespaces & cgroups). Isliye container fast aur lightweight hota hai.

```
VM Architecture:                    Docker Architecture:
┌─────────┬─────────┐              ┌─────────┬─────────┐
│  App A  │  App B  │              │  App A  │  App B  │
│ Guest OS│ Guest OS│              │Container│Container│
├─────────┴─────────┤              ├─────────┴─────────┤
│    Hypervisor      │              │   Docker Engine    │
├─────────────────────┤              ├─────────────────────┤
│     Host OS         │              │     Host OS         │
├─────────────────────┤              ├─────────────────────┤
│     Hardware         │              │     Hardware         │
└─────────────────────┘              └─────────────────────┘
```

---

## 3. Docker Architecture (Client-Server Model)

Docker ka architecture 3 main parts se bana hai — **yeh diagram bana ke bologe interview mein, impact padega:**

1. **Docker Client** — Jab tum terminal mein `docker run` type karte ho, yeh Client hai. Yeh tumhare commands leta hai.
2. **Docker Daemon (dockerd)** — Yeh background mein chalta hai, actual kaam karta hai — images build karna, containers run/stop karna, networks/volumes manage karna. Client, Daemon ko REST API ke through commands bhejta hai.
3. **Docker Registry** — Yahan images store hoti hain (jaise Docker Hub — GitHub jaisa hi hai, but images ke liye).

```
docker run nginx
     │
     ▼
┌──────────────┐    REST API    ┌──────────────┐
│ Docker Client │ ─────────────► │ Docker Daemon │
└──────────────┘                └───────┬───────┘
                                          │
                          ┌───────────────┼───────────────┐
                          ▼               ▼               ▼
                     Images          Containers       Networks/Volumes
                          │
                          ▼
                  ┌───────────────┐
                  │ Docker Registry│ (Docker Hub)
                  └───────────────┘
```

**Interview line:** "Docker ek client-server architecture follow karta hai. Client daemon ko command bhejta hai, daemon images build/run karta hai, aur registry (Docker Hub) images store karta hai."

---

## 4. Image vs Container (Sabse Confusing Concept — Clear Karte Hain)

Yeh analogy yaad rakho: **Class aur Object jaisa relationship hai (agar tumne OOP padha hai — tumne padha hai!).**

- **Image** = Class → Blueprint/Template. Read-only. Isme instructions hoti hain ki container kaise banega (kaunsa OS, kaunsi dependencies, kaunsa code).
- **Container** = Object → Image ka **running instance**. Ek image se multiple containers bana sakte ho.

```
Image (Blueprint)  ──run──►  Container 1 (Running)
     python:3.11    ──run──►  Container 2 (Running)
                     ──run──►  Container 3 (Running)
```

**Interview line:** "Image ek immutable template hai jisme application code aur dependencies packaged hain. Container us image ka executable, running instance hai. Ek image se multiple independent containers spawn ho sakte hain."

---

## 5. Basic Docker Commands (Yeh Sab Ratt Lo, Coding Round Mein Kaam Aayenge)

```bash
# Image se related
docker pull python:3.11          # Docker Hub se image download karo
docker images                    # Saari local images list karo
docker rmi <image_id>            # Image delete karo
docker build -t myapp:v1 .       # Current folder ke Dockerfile se image banao

# Container se related
docker run myapp:v1              # Image se naya container banao aur chalao
docker run -d myapp:v1           # Detached mode (background mein chalega)
docker run -p 8000:8000 myapp:v1 # Port mapping: host:container
docker run -it myapp:v1 bash     # Interactive mode + terminal access
docker ps                        # Running containers dikhao
docker ps -a                     # Saare containers dikhao (stopped bhi)
docker stop <container_id>       # Container stop karo
docker start <container_id>      # Stopped container start karo
docker rm <container_id>         # Container delete karo
docker logs <container_id>       # Container ke logs dekho
docker exec -it <container_id> bash  # Running container ke andar jaao

# Cleanup
docker system prune              # Unused images/containers/networks clean karo
```

**Interview mein pooch sakte hain:** *"docker run aur docker start mein kya difference hai?"*
> `docker run` = naya container **create** karta hai image se, aur start karta hai.
> `docker start` = already existing (stopped) container ko **restart** karta hai.

---

## 6. Dockerfile — Poori Detail Mein (Sabse Important Section)

Dockerfile ek text file hai jisme instructions hoti hain ki image kaise banegi. Chalo ek **FastAPI app ka real Dockerfile** samajhte hain (tumhare current prep se directly connect hoga):

Pehle tumhara normal FastAPI project structure socho:
```
myproject/
├── main.py
├── requirements.txt
└── Dockerfile
```

`requirements.txt`:
```
fastapi
uvicorn
```

`main.py`:
```python
from fastapi import FastAPI
app = FastAPI()

@app.get("/")
def read_root():
    return {"message": "Docker ke saath FastAPI chal raha hai!"}
```

Ab **Dockerfile** banate hain, line by line samjhte hue:

```dockerfile
# 1. Base image — kis OS/language version pe banega container
FROM python:3.11-slim

# 2. Container ke andar working directory set karo
WORKDIR /app

# 3. requirements.txt ko pehle copy karo (caching ke liye — niche samjhaya hai)
COPY requirements.txt .

# 4. Dependencies install karo
RUN pip install --no-cache-dir -r requirements.txt

# 5. Baaki saara code copy karo
COPY . .

# 6. Batao container kaunsa port use karega (documentation purpose)
EXPOSE 8000

# 7. Container start hote hi yeh command chalegi
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

**Build aur run karne ke commands:**
```bash
docker build -t fastapi-app .
docker run -d -p 8000:8000 fastapi-app
```

Ab browser mein `localhost:8000` khol ke check kar sakte ho — response aayega!

### Har Instruction Ka "Why" (Interview mein yeh depth dikhao)

| Instruction | Kaam Kya Karta Hai |
|---|---|
| `FROM` | Base image define karta hai (har Dockerfile ki pehli line hoti hai) |
| `WORKDIR` | Container ke andar ek folder banata hai aur usme "cd" kar deta hai |
| `COPY` | Host machine se container mein files copy karta hai |
| `RUN` | Build-time pe command execute karta hai (jaise install karna) — **image mein permanent layer bana deta hai** |
| `CMD` | Container **start hone ke time** default command run karta hai |
| `EXPOSE` | Documentation ke liye batata hai container kis port pe listen karega (actual port mapping `-p` flag se hoti hai) |
| `ENV` | Environment variable set karta hai |
| `ARG` | Build-time variable (jaise build karte waqt version pass karna) |

### ⭐ Requirements.txt pehle copy karne ka trick (Interview mein zaroor poochhte hain — "Docker Layer Caching")

Docker image **layers** mein bani hoti hai. Har instruction (COPY, RUN, etc.) ek naya layer banata hai. Agar koi layer nahi badla, Docker **cache use kar leta hai** — rebuild fast ho jata hai.

Isliye humne pehle `requirements.txt` copy kiya, phir `pip install` chalaya, **uske baad** pura code copy kiya. Agar sirf `main.py` change hua (dependencies nahi), toh Docker `pip install` wala layer **cache se le lega**, dobara install nahi karega. Agar humne seedha `COPY . .` pehle kar diya hota, toh har chhote code change pe **saari dependencies dobara install hoti** — slow build.

> **Interview line:** "Docker image layers mein banti hai, aur caching layer-wise hoti hai. Isliye humesha wo files pehle copy karo jo kam badalti hain (jaise requirements.txt), taaki dependency installation cache se reuse ho sake aur build time fast rahe."

---

## 7. CMD vs ENTRYPOINT (Bahut Common Interview Trap Question)

Dono hi container start hone pe command run karte hain, but difference hai:

- **CMD** — Default command hai, jo **override ho sakta hai** jab tum `docker run` mein extra arguments do.
- **ENTRYPOINT** — Fixed command hai, jo override **nahi hota** (extra arguments CMD/ENTRYPOINT ke saath append hote hain).

```dockerfile
CMD ["uvicorn", "main:app"]
```
Agar tum `docker run myimage python other.py` chalao, toh CMD **poora override** ho jayega — `uvicorn` wali command nahi chalegi.

```dockerfile
ENTRYPOINT ["uvicorn"]
CMD ["main:app"]
```
Yahan `uvicorn` fixed hai, `main:app` default argument hai jo override ho sakta hai. Agar `docker run myimage other:app` chalao, toh final command banegi: `uvicorn other:app`.

**Interview line:** "ENTRYPOINT container ka main executable define karta hai jo change nahi hota, jabki CMD default arguments deta hai jo runtime pe override kiye ja sakte hain."

---

## 8. COPY vs ADD

- `COPY` — Simple file/folder copy karta hai. **Best practice: hamesha COPY use karo.**
- `ADD` — COPY jaisa hi, plus extra features: URL se download kar sakta hai, aur `.tar` files ko auto-extract kar deta hai.

**Interview line:** "COPY simple aur predictable hai, isliye best practice mein prefer kiya jaata hai. ADD extra magic karta hai (auto-extraction, URL download) jo kabhi-kabhi unexpected behavior de sakta hai."

---

## 9. Docker Volumes — Data Persistence (Bahut Important Concept)

**Problem:** Container delete hote hi uske andar ka data **gayab ho jaata hai** (containers stateless hote hain by design).

**Solution:** Volumes — yeh host machine pe data store karte hain, container delete hone ke baad bhi data safe rehta hai.

```bash
# Named volume banake use karo (recommended)
docker run -v mydata:/app/data myapp

# Bind mount — host ka specific folder container se connect karo
docker run -v /home/prathamesh/data:/app/data myapp
```

| Type | Kya Hai | Kab Use Karo |
|---|---|---|
| **Volume** | Docker khud manage karta hai (`/var/lib/docker/volumes`) | Production mein — safe, portable |
| **Bind Mount** | Tum khud host ka path specify karte ho | Development mein — live code reload ke liye |

**Interview line:** "Containers by default stateless hote hain — restart/delete pe data lost ho jaata hai. Volumes container ke data ko host machine pe persist karte hain, taaki database jaisi cheezein safe rahein."

---

## 10. Docker Networking

Docker mein 3 main network types hain:

1. **Bridge** (default) — Container ek isolated internal network pe chalte hain, host ke saath port mapping (`-p`) se connect hote hain.
2. **Host** — Container directly host ki networking use karta hai, no isolation.
3. **None** — Koi networking nahi, fully isolated.

Jab tum **Docker Compose** use karte ho, toh sab containers automatically ek common bridge network pe aa jaate hain, aur ek container doosre ko **service name se hi call** kar sakta hai (jaise `http://db:5432`) — DNS jaisa kaam karta hai internally.

---

## 11. Docker Compose — Multiple Containers Ek Saath (High Priority for Interviews)

Real projects mein sirf ek container nahi hota — FastAPI + Database + Redis, sab saath chalte hain. **Docker Compose** isko manage karta hai ek single YAML file se.

`docker-compose.yml` (FastAPI + PostgreSQL example):

```yaml
version: '3.8'

services:
  api:
    build: .
    ports:
      - "8000:8000"
    depends_on:
      - db
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/mydb

  db:
    image: postgres:15
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=mydb
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

Run karne ka command:
```bash
docker-compose up -d       # sab services start karo (background mein)
docker-compose down        # sab band karo
docker-compose logs -f api # ek specific service ke logs dekho
```

**Notice karo:** `api` service ke andar humne `db:5432` likha, `localhost` nahi — kyuki Compose automatically dono containers ko ek network pe daal deta hai aur service names DNS ki tarah kaam karte hain.

**Interview line:** "Docker Compose multi-container applications ko ek YAML file se define aur orchestrate karta hai. Har service apna container hai, aur Compose automatically inhe ek common network pe connect kar deta hai, jisse services ek-doosre ko service name se access kar sakti hain."

---

## 12. Multi-Stage Builds (Advanced — Senior Interviewers Yeh Bhi Poochhte Hain)

Problem: Build tools (compilers, etc.) final image mein nahi chahiye — sirf final app chahiye. Multi-stage build se image ka size bahut kam ho jaata hai.

```dockerfile
# Stage 1: Build stage
FROM python:3.11 AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user -r requirements.txt

# Stage 2: Final lightweight stage
FROM python:3.11-slim
WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY . .
CMD ["uvicorn", "main:app", "--host", "0.0.0.0"]
```

**Interview line:** "Multi-stage builds mein hum multiple FROM statements use karte hain — pehla stage dependencies build karta hai, doosra stage sirf zaroori files copy karta hai. Isse final image bahut chhoti aur secure banti hai, kyuki build tools final image mein nahi jaate."

---

## 13. Rapid-Fire Interview Q&A (Yeh Zaroor Ratt Lo)

**Q1: Docker container aur image mein basic difference?**
> Image = static blueprint. Container = image ka running instance.

**Q2: Container kaise isolated rehta hai agar OS same hai?**
> Linux ke do features: **Namespaces** (process, network, filesystem isolation) aur **cgroups** (resource limiting — CPU, memory).

**Q3: Docker Hub kya hai?**
> Docker images ke liye public/private registry — jaise GitHub hai code ke liye.

**Q4: Ek container ke andar multiple processes chala sakte ho?**
> Technically ho sakta hai, but **best practice** hai — "one process per container". Isse scaling aur debugging aasan hoti hai.

**Q5: .dockerignore kya hota hai?**
> `.gitignore` jaisa hi — files/folders (jaise `__pycache__`, `.git`, `venv/`) ko image build context se exclude karta hai, build fast hota hai aur image size chhoti rehti hai.

**Q6: Docker container restart hone pe data kya hota hai?**
> Agar volume mount nahi kiya toh data **lost** ho jaata hai. Volume mount kiya ho toh data persist rehta hai.

**Q7: docker stop vs docker kill?**
> `stop` — graceful shutdown (SIGTERM signal bhejta hai, thoda time deta hai)
> `kill` — immediately force stop (SIGKILL)

**Q8: Image ka size kaise reduce karoge?**
> Slim/alpine base images use karo, multi-stage builds karo, unnecessary layers combine karo (`RUN` commands ko `&&` se chain karo), `.dockerignore` use karo.

**Q9: Container aur process mein kya relation hai?**
> Container basically host OS pe ek **isolated process** hi hota hai — bas namespaces/cgroups se isolate kiya gaya.

**Q10: Docker mein ENV variable kaise pass karte ho?**
```bash
docker run -e DATABASE_URL=postgres://... myapp
```
ya Dockerfile mein `ENV KEY=value`.

---

## 14. Ek Line Mein Poori Docker Journey (Interview Summary Ke Liye)

> "Maine Dockerfile likha jisme FastAPI app ke liye Python base image use ki, dependencies install ki layer caching ka fayda uthate hue, phir `docker build` se image banayi. Multi-container setup (FastAPI + Postgres) ke liye Docker Compose use kiya, jisme services automatically internal network pe connect ho jaati hain. Data persistence ke liye volumes use kiye, aur production ke liye multi-stage build se image size optimize kiya."

Yeh ek line bologe interview mein toh interviewer ko turant pata chal jaayega ki tumne sirf theory nahi, **practically Docker use kiya hai.**

---

## 15. Ab Practice Karo (Action Items)

1. Apne kisi existing project (FastAPI ya MedIntel) ko Dockerize karo — upar wala Dockerfile template use karo.
2. `docker build`, `docker run`, `docker ps`, `docker logs` commands khud chalao.
3. Ek `docker-compose.yml` banao jisme apna app + koi database ho.
4. GitHub pe Dockerfile push karo — resume mein ek line add ho jaayegi: *"Containerized [project name] using Docker for consistent deployment across environments."*

Agar chaho toh main tumhare **MedIntel AI ya BharatNum project ko actually Dockerize karke** step-by-step dikha sakta hoon — real hands-on practice ho jaayegi, aur GitHub pe bhi daal sakte ho. Bolo agar karna hai.
