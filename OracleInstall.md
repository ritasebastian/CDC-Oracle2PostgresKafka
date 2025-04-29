**complete guide** from **Docker installation** to **running Oracle DB** and **enabling ARCHIVELOG mode** — all combined nicely.

---
# 🐳💾 Full Guide: Install Docker, Run Oracle DB, Enable ARCHIVELOG Mode (MacBook Air)

---

# ✅ Step 1: Install Docker Desktop on Mac

### 1. Download Docker Desktop:
- Go to: [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)
- Select **Mac with Intel chip** or **Mac with Apple chip** depending on your MacBook (Intel vs M1/M2/M3).

### 2. Install Docker:
- Open the downloaded `.dmg` file
- Drag and drop Docker into Applications
- Launch Docker from Applications
- Wait until 🐳 **Docker Desktop is running** (Whale icon on top bar)

---

# ✅ Step 2: Pull Oracle Database Docker Image

Oracle provides official database images now.

### 1. Login to Oracle Container Registry
- Visit: [https://container-registry.oracle.com](https://container-registry.oracle.com)
- Sign in with your Oracle account (create free if you don't have)
- Accept license for:
  - **Database** → **Free** (or Enterprise if you want)

> You need to accept license **one time** before you can pull images.

---

### 2. Pull Oracle Free Database Image
Now in Terminal, pull the Oracle DB Free image:

```bash
docker login container-registry.oracle.com
```
(enter Oracle username and password)

Then pull the image:

```bash
docker pull container-registry.oracle.com/database/free:23.3.0
```
```bash
docker pull gvenzl/oracle-xe
```
✅ This will download the lightweight Oracle 23c Free version (~2-3 GB).

---

# ✅ Step 3: Run Oracle Database Container

Now run the Oracle DB container:

```bash
docker run -d --name oracle-free \
  -p 1521:1521 -p 5500:5500 \
  -e ORACLE_PWD=Oracle123 \
  container-registry.oracle.com/database/free:23.3.0
```
```bash
docker run -d --name oracle-xe \
  -p 1521:1521 \
  -e ORACLE_PASSWORD=Oracle123 \
  gvenzl/oracle-xe
```
| Setting  | Value |
|----------|-------|
| Container Name | `oracle-free` |
| Oracle password | `Oracle123` |
| Ports | 1521 (SQL*Net), 5500 (EM Express) |

✅ Oracle DB will start inside the container.

---

# ✅ Step 4: Enable ARCHIVELOG Mode

Now let's **enable ArchiveLog mode** inside the running container:

---

### 1. Enter into the container
```bash
docker exec -it oracle-free /bin/bash
```

---

### 2. Connect to SQL*Plus as SYSDBA
```bash
sqlplus / as sysdba
```

---

### 3. Check Current Archive Log Mode
```sql
archive log list;
```
- You will see: **No Archive Mode**

---

### 4. Shutdown Database Cleanly
```sql
shutdown immediate;
```

---

### 5. Start Database in MOUNT mode
```sql
startup mount;
```

---

### 6. Enable ArchiveLog
```sql
alter database archivelog;
```

---

### 7. Open the Database
```sql
alter database open;
```

---

### 8. Confirm it
```sql
archive log list;
```
✅ You should see **Archive Mode enabled**!

---

# 🛠 Extra Tips:

- **Connect from your Mac SQL tools** (like DBeaver, Oracle SQL Developer):
  - Host: `localhost`
  - Port: `1521`
  - Service Name: `FREEPDB1`
  - Username: `system`
  - Password: `Oracle123`
  
- **Monitor archive logs** inside container (default location):
  ```bash
  /opt/oracle/oradata/ORCLCDB/archivelogs
  ```

- **Clean up container** if needed:
  ```bash
  docker stop oracle-free
  docker rm oracle-free
  ```

---

