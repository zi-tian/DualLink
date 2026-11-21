# 设备固件 OTA 协议（DualLink）

## Flash 分区（STC8H8K64U，64KB，扇区 512 字节）

| 地址范围 | 大小 | 用途 |
|---|---|---|
| 0x0000 - 0x4FFF | 20KB | 应用区（正常运行） |
| 0x5000 - 0x9FFF | 20KB | 下载区（App 传上来的新固件） |
| 0xA000 - 0xA1FF | 512B | 标志与版本（升级中 / 待搬运） |
| 0xF000 - 0xFFFF | 4KB | Boot 搬运程序 |

## 命令

| 命令 | 应答 |
|---|---|
| `OTA?` | `OTA idle` / `OTA ready 0x1200` / `OTA busy` |
| `OTA:BEGIN,<总字节>,<CRC16>` | `OK OTA BEGIN <总字节>` |
| `OTA:DATA,<序号>,<HEX>` | `OK OTA DATA <序号>` / `ERR OTA CRC` |
| `OTA:END` | `OK OTA READY` 然后自动跳 Boot 搬运 |
| `OTA:ABORT` | `OK OTA ABORT` |
| `VER?` | `VER app=2.1 boot=1.0` |

约定：

- 单行长度不超过 95 字节（模组行缓冲 96），所以每包 32 字节数据（64 个 hex 字符）
- 序号从 0 开始，设备按序号写对应偏移；重发同序号会覆盖写入
- CRC16 用 Modbus 多项式（0xA001，初值 0xFFFF），覆盖整段固件数据

## 流程

1. App 发 `OTA:BEGIN,<size>,<crc>`，设备擦除下载区
2. App 按 32 字节分包发 `OTA:DATA,<seq>,<hex>`，收到 `OK OTA DATA` 再发下一包
3. 全部发完发 `OTA:END`，设备复核 CRC，写“待搬运”标志，跳到 0xF000
4. Boot 校验 → 擦应用区 → 逐扇区搬运 → 复核 → 清标志 → 跳 0x0000
5. 搬运中断电：上电后标志仍为“搬运中”且下载区 CRC 正确，Boot 自动重搬一次

## 失败处理

- 下载阶段失败：`OTA:ABORT` 清状态，应用区未动，设备照常运行
- 搬运阶段失败：设备会停在 Boot，用 STC-ISP 重新烧录合并后的 hex 即可
