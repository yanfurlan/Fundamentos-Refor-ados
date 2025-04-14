**Fundamentos Reforçados**

---

# 🏦 1. Arquitetura Oracle

## 1.1 Componentes da Arquitetura
- **SGA (System Global Area)**:
  - Shared Pool (Library Cache, Data Dictionary Cache)
  - Database Buffer Cache
  - Redo Log Buffer
  - Large Pool
  - Java Pool
  - Streams Pool
- **PGA (Program Global Area)**:
  - Area privada por processo (Hash Joins, Sorts, Bitmap Merge)
- **Background Processes**:
  - **DBWn**: Database Writer
  - **LGWR**: Log Writer
  - **CKPT**: Checkpoint
  - **SMON**: System Monitor
  - **PMON**: Process Monitor
  - **RECO**: Recoverer (para Distributed Databases)
  - **ARCn**: Archiver (modo ARCHIVELOG)
  - **MMON / MMNL**: Monitoramento automátizado (AWR, ADDM)

## 1.2 Arquivos Essenciais
- **Datafiles**: Armazenam dados persistentes
- **Redo Logs**: Garantem integridade das transações
- **Control Files**: Arquivos de controle com metadados do banco
- **Parameter Files**:
  - **SPFILE** (Server Parameter File) - binário
  - **PFILE** (Parameter File) - texto

---

# ⚖️ 2. Estrutura Lógica e Física

## 2.1 Estrutura Lógica
- **Tablespaces**: Lógicas, contêm segmentos
- **Segments**: Tabelas, índices, undo, etc.
- **Extents**: Agrupamentos contíguos de blocos
- **Blocos (DB Blocks)**: Menor unidade de armazenamento (padrão: 8KB)

## 2.2 Estrutura Física
- **Arquivos de Dados** (.dbf)
- **Arquivos de Redo Log**
- **Arquivos de Controle**
- **Arquivos Temporários**

## 2.3 Tipos de Tablespaces
- **Permanent**: SYSTEM, SYSAUX, USERS
- **Temporary**: TEMP, TEMPFILE
- **Undo**: Armazena undo data
- **Bigfile vs Smallfile**

---

# 🔐 3. Gerenciamento de Usuários, Acessos e Segurança

## 3.1 Criação de Usuários
```sql
CREATE USER nome IDENTIFIED BY senha;
GRANT CONNECT, RESOURCE TO nome;
ALTER USER nome DEFAULT TABLESPACE users;
```

## 3.2 Roles e Privilégios
- **System Privileges**: CREATE SESSION, CREATE TABLE, etc.
- **Object Privileges**: SELECT, INSERT, UPDATE, DELETE
- **Roles Customizadas**:
```sql
CREATE ROLE dba_readonly;
GRANT SELECT ON tabela TO dba_readonly;
```

## 3.3 Profiles
- Limita recursos como tempo de CPU, conexões, falhas de login
- Política de senhas, validade, histórico

## 3.4 Segurança Avançada
- **Auditoria FGA** (Fine Grained Auditing)
- **Oracle Label Security**
- **Transparent Data Encryption (TDE)**

---

# 💾 4. Backup e Recovery com RMAN

## 4.1 Tipos de Backup
- **Backup Físico** (RMAN)
- **Backup Lógico** (Data Pump - expdp/impdp)
- **Hot Backup** (com banco online)
- **Cold Backup** (banco offline)

## 4.2 Comandos Essenciais
```sql
BACKUP DATABASE;
BACKUP AS COMPRESSED BACKUPSET DATABASE;
BACKUP INCREMENTAL LEVEL 0 DATABASE;
RESTORE DATABASE;
RECOVER DATABASE;
```

## 4.3 Configuração
```sql
CONFIGURE RETENTION POLICY TO REDUNDANCY 2;
CONFIGURE CONTROLFILE AUTOBACKUP ON;
CONFIGURE CHANNEL DEVICE TYPE DISK FORMAT '/backup/rman_%U.bkp';
```

## 4.4 Fast Recovery Area (FRA)
- Área dedicada para armazenar arquivos de recuperação
- Local configurado via `DB_RECOVERY_FILE_DEST`

## 4.5 Recovery Catalog
- Banco separado com metadados históricos de backup RMAN
- Permite restores mais complexos e histórico de backup

---

# ⏱ 5. Agendamento de Jobs

## 5.1 DBMS_SCHEDULER
```sql
BEGIN
  DBMS_SCHEDULER.CREATE_JOB(
    job_name => 'job_backup',
    job_type => 'PLSQL_BLOCK',
    job_action => 'BEGIN BACKUP DATABASE; END;',
    start_date => SYSTIMESTAMP,
    repeat_interval => 'FREQ=DAILY;BYHOUR=2',
    enabled => TRUE
  );
END;
```

## 5.2 Monitoramento
```sql
SELECT * FROM DBA_SCHEDULER_JOBS;
SELECT * FROM DBA_SCHEDULER_JOB_RUN_DETAILS;
```

## 5.3 Chains, Events, Programs
- Composição de tarefas mais complexas (como workflows)

---

# ⚙️ 6. Diagnóstico e Tuning Básico

## 6.1 Views Importantes
- **V$INSTANCE**
- **V$DATABASE**
- **V$SESSION** / **V$PROCESS**
- **V$SQL**, **V$SQLAREA**, **V$SQL_PLAN**
- **DBA_HIST_SQLSTAT**, **DBA_HIST_ACTIVE_SESS_HISTORY**

## 6.2 Ferramentas
- **AWR (Automatic Workload Repository)**
- **ASH (Active Session History)**
- **ADDM (Automatic Database Diagnostic Monitor)**
- **SQL Tuning Advisor** / **SQL Access Advisor**

---

# 📦 7. Exadata Fundamentals

## 7.1 Conceitos
- **Exadata Storage Server (Cell Server)**
- **Smart Scan**: Filtragem de dados no storage
- **Hybrid Columnar Compression (HCC)**
- **Flash Cache** e **Smart Flash Logging**
- **IORM (I/O Resource Manager)**: Gerenciamento de I/O entre bancos

## 7.2 Configurações Típicas
- RAC + ASM + Grid Infrastructure
- Monitoramento via **Enterprise Manager** ou **dcli/cellcli**

---

# 📌 8. ASM (Automatic Storage Management)

## 8.1 Conceito
- Substitui volume managers do SO
- Gerencia discos de forma automática

## 8.2 Componentes
- **Disk Groups**: AGRUPAM discos
- **Redundância**: Normal, High, External

## 8.3 Comandos
```sql
CREATE DISKGROUP data NORMAL REDUNDANCY
  DISK '/dev/sdX1', '/dev/sdX2'
  ATTRIBUTE 'compatible.asm' = '19.0';
```

---

# 🌐 9. RAC (Real Application Clusters)

## 9.1 Objetivo
- Alta disponibilidade e escalabilidade horizontal
- Vários nós acessam a mesma base

## 9.2 Componentes
- **Clusterware**
- **CRS** (Cluster Ready Services)
- **SCAN Listener**
- **Cache Fusion**: Compartilhamento de buffers entre nós

---

# ✨ 10. Boas Práticas Reforçadas

- Separar arquivos de dados, redo e backups em dispositivos diferentes
- Monitorar uso de PGA/SGA com AWR
- Utilizar RMAN com catalog e rotina automatizada
- Controlar privilégios com roles e profiles restritos
- Realizar testes de restore a cada trimestre
- Atualizar patches críticos com OPatch

---

# 📚 Referências
- Oracle Documentation: https://docs.oracle.com
- Oracle Exadata Documentation: https://docs.oracle.com/en/engineered-systems/exadata/
- Oracle LiveLabs: https://developer.oracle.com/livelabs
- Livro "Oracle Database 19c Handbook" (Oracle Press)
- Blog Oracle Base: https://oracle-base.com
- Oracle Learning Library

---
