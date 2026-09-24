# APSX Platform — Claude plugin marketplace

โฟลเดอร์นี้คือ **plugin marketplace** สำหรับลูกค้าที่ใช้โปรแกรมคลินิก APSX Platform
ออกแบบให้ push เป็น git repo แยก (เช่น `github.com/apsth/claude-plugins`) หรือ zip ขึ้น S3 ก็ได้
ตัว backend ของ plugin คือ MCP server ใน `shopapiv2` (`/mcp` + `/oauth/*`)

## โครงสร้าง

```
claude-plugin/
├── .claude-plugin/marketplace.json     ← รายการ plugin (ชื่อ marketplace: apsx-platform)
├── plugins/apsx-clinic/
│   ├── .claude-plugin/plugin.json      ← manifest
│   ├── .mcp.json                       ← ชี้ MCP server (OAuth ผ่านหน้า login ของ APSX)
│   └── skills/apsx-clinic/SKILL.md     ← สอน Claude ใช้ tool + กติกา PDPA
└── README.md
```

## ลูกค้าติดตั้งอย่างไร

### Claude Code (CLI / VS Code / Desktop)

```
/plugin marketplace add apsth/claude-plugins        # หลัง push repo นี้ขึ้น GitHub
/plugin install apsx-clinic@apsx-platform
/mcp                                                # เลือก apsx-clinic → เบราว์เซอร์เปิดหน้า login ของ APSX
```

ทดสอบจากโฟลเดอร์นี้โดยตรง (ยังไม่ต้อง push):

```
/plugin marketplace add ./claude-plugin
/plugin install apsx-clinic@apsx-platform
```

ชี้ไป server อื่น (เช่น dev) ด้วย env ก่อนเปิด Claude Code: `export APSX_MCP_URL=http://127.0.0.1:8099/mcp`

### claude.ai / Claude Desktop (ไม่ต้องมี plugin)

Settings → Connectors → **Add custom connector** → ใส่ URL `https://shopv2.api-apsx.com/mcp`
→ กด Connect → ล็อกอินด้วยบัญชีโปรแกรมคลินิก (ใช้ได้ทุก plan · Free จำกัด 1 connector)

### องค์กร (Team/Enterprise)

Admin Settings → Plugins → เพิ่ม marketplace นี้ → แจกให้สมาชิกใน org ได้เอง
(หน้า "Create a plugin" ใน claude.ai แจกได้เฉพาะคนใน org เดียวกัน — ลูกค้าคนละ org ต้องใช้ marketplace)

## ฝั่ง server ต้องมีอะไร

| อย่าง | ค่า |
|---|---|
| migration | 253 (`clinic_main_x`) + 254 (`clinic_log_x`) **ก่อน deploy** |
| env | `MCP_ISSUER_URL=https://shopv2.api-apsx.com` (URL สาธารณะ · ไม่ตั้ง = เดาจาก Host header) · `MCP_ACCESS_TTL_MIN` (default 60) · `MCP_REFRESH_TTL_HOURS` (default 720) |
| endpoint | `GET /.well-known/oauth-authorization-server` · `GET /.well-known/oauth-protected-resource` · `POST /oauth/register` · `GET/POST /oauth/authorize` · `POST /oauth/token` · `POST /oauth/revoke` · `POST /mcp` |

รายละเอียดเต็ม: `Clinic-APP/_meta/mcp-server-claude-plugin-2026-09-24.md`

## เวอร์ชัน

| version | วันที่ | เปลี่ยนอะไร |
|---|---|---|
| 1.1.0 | 2026-09-24 | เพิ่มชุด "วิเคราะห์ธุรกิจ" 7 tool (`sales_by_category` · `best_sellers` · `staff_earnings` · `appointment_summary` · `revenue_by_period` · `payment_summary` · `outstanding_balances`) + กติกาการอ่านผลใน SKILL.md · ต้องใช้กับ `shopapiv2` ที่ deploy ≥ 2026-09-24 |
| 1.0.0 | 2026-09-24 | เวอร์ชันแรก 24 tool อ่านอย่างเดียว + OAuth ผ่านหน้า login ของ APSX |

> ลูกค้าที่ติดตั้ง plugin ไว้แล้ว: `claude plugin update apsx-clinic` (หรือถอน/ติดตั้งใหม่) เพื่อรับ SKILL.md ใหม่ · ผู้ใช้ claude.ai connector ไม่ต้องทำอะไร แค่ Reconnect ให้โหลดรายการ tool ใหม่
