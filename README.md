# วิธีตั้งค่า n8n Agentic RAG Agent

โปรเจกต์นี้เป็นตัวอย่าง workflow สำหรับทำ RAG Agent บน n8n โดยใช้ Google Drive เป็นแหล่งไฟล์เอกสาร และใช้ Supabase/PostgreSQL พร้อม pgvector สำหรับเก็บ vector database

## สิ่งที่ต้องเตรียม

- Node.js และ npm
- บัญชี n8n หรือ n8n ที่รันบนเครื่อง
- บัญชี Google Cloud สำหรับสร้าง OAuth ของ Google Drive
- โปรเจกต์ Supabase สำหรับฐานข้อมูล PostgreSQL
- API key ของโมเดล/embedding ที่ใช้ใน workflow

## 1. ติดตั้ง n8n

ติดตั้ง n8n แบบ global ด้วย npm

```bash
npm install -g n8n
```

ตรวจสอบเวอร์ชัน

```bash
n8n -v
```

อัปเดต n8n เมื่อต้องการใช้เวอร์ชันล่าสุด

```bash
npm update -g n8n
```

## 2. รัน n8n

เริ่มใช้งาน n8n บนเครื่อง

```bash
n8n start
```

จากนั้นเปิดหน้า n8n ตาม URL ที่แสดงใน terminal โดยทั่วไปจะเป็น

```text
http://localhost:5678
```

## 3. ติดตั้ง Community Node

ใน n8n ให้ติดตั้ง node เพิ่มเติมสำหรับ Gemini embedding

```text
n8n-nodes-gemini-embedding-plus
```

วิธีติดตั้งใน n8n:

1. ไปที่ Settings
2. เลือก Community nodes
3. เลือก Install
4. ใส่ชื่อ package `n8n-nodes-gemini-embedding-plus`
5. กด Install

## 4. ตั้งค่า Google Drive OAuth

ใช้สำหรับให้ workflow เข้าถึงไฟล์ใน Google Drive

1. ไปที่ Google Cloud Console: https://console.cloud.google.com/
2. เลือกหรือสร้างโปรเจกต์ใหม่
3. ไปที่ APIs & Services > Library
4. เปิดใช้งาน Google Drive API
5. ไปที่ APIs & Services > Credentials
6. สร้าง OAuth 2.0 Client ID
7. คัดลอก Client ID และ Client Secret
8. นำค่าไปใส่ใน credential ของ Google Drive บน n8n

หมายเหตุ: ถ้าใช้ n8n บนเครื่อง ให้ตรวจสอบ Redirect URI ในหน้า credential ของ n8n แล้วนำไปใส่ใน Google Cloud Console ให้ตรงกัน

## 5. ตั้งค่า Supabase และ PostgreSQL

ใช้ Supabase เป็นฐานข้อมูลสำหรับเก็บเอกสารและ embedding

1. สร้างโปรเจกต์ใน Supabase
2. ไปที่ Project Settings > API
3. คัดลอก `service_role` key หรือ key ที่ workflow ต้องใช้
4. ไปที่ Connect เพื่อดู connection string ของ PostgreSQL
5. เลือกแบบ Session Pooler หากต้องการเชื่อมต่อผ่าน pooler
6. นำค่า host, database, user, password และ port ไปตั้งค่าใน credential ของ PostgreSQL บน n8n

## 6. สร้างตาราง vector store

เปิด SQL Editor ใน Supabase แล้วรันไฟล์นี้

```text
vectorStoreSQL.sql
```

ไฟล์นี้จะสร้าง:

- extension `vector`
- ตาราง `documents`
- function `match_documents`

ถ้าเปลี่ยนโมเดล embedding ให้ตรวจสอบขนาด vector ในไฟล์ SQL ด้วย เช่น `extensions.vector(768)` ต้องตรงกับ dimension ของ embedding model ที่ใช้

## 7. Import workflow ตัวอย่าง

นำไฟล์ workflow เข้า n8n จากเมนู Import from file

```text
example/RAG-Agent-Example.json
example/RAG-VectorStore-Example.json
```

หลัง import แล้วให้ตรวจสอบ credential ในแต่ละ node เช่น

- Google Drive
- Supabase/PostgreSQL
- Gemini หรือ embedding model
- Chat model ที่ใช้ตอบคำถาม

## 8. เริ่มใช้งาน

ลำดับการใช้งานโดยทั่วไป:

1. รัน workflow สำหรับนำเข้าเอกสารและสร้าง vector store ก่อน
2. ตรวจสอบว่าเอกสารถูกบันทึกลงตาราง `documents` ใน Supabase แล้ว
3. เปิดใช้งาน workflow ของ RAG Agent
4. ทดลองถามคำถามจากเอกสารที่อยู่ใน Google Drive

## ไฟล์สำคัญในโปรเจกต์

- `README.md` คู่มือการตั้งค่า
- `vectorStoreSQL.sql` SQL สำหรับสร้าง vector store ใน Supabase
- `example/RAG-Agent-Example.json` workflow ตัวอย่างสำหรับ RAG Agent
- `example/RAG-VectorStore-Example.json` workflow ตัวอย่างสำหรับสร้าง vector store
- `translate.json` ไฟล์แปลหรือข้อมูลประกอบเพิ่มเติม

## หมายเหตุ

- ห้ามเผยแพร่ Client Secret, API key หรือ `service_role` key ลง GitHub
- ถ้า workflow หาเอกสารไม่เจอ ให้ตรวจสอบ permission ของ Google Drive และ credential ใน n8n
- ถ้าค้นหาเอกสารแล้วผลลัพธ์ไม่ตรง ให้ตรวจสอบ dimension ของ embedding และข้อมูลในตาราง `documents`
