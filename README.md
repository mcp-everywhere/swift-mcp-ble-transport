# SwiftMCPBLETransport

`SwiftMCPBLETransport` 是一个面向 iOS 的 BLE 传输库，用于把 `modelcontextprotocol/swift-sdk` 的 `Client` 连接到 BLE MCP 设备。

## 安装

### 平台与工具链

- 本包声明支持 iOS 16+、macOS 13+；BLE 连接依赖 CoreBluetooth，应用还需配置下文的蓝牙权限。
- 本包的 `swift-tools-version` 为 6.0，但依赖的 `modelcontextprotocol/swift-sdk` 0.12.1 要求 Swift 6.1+，因此不能仅按本包清单判断所需工具链。
- 当前仓库的 `Package.resolved` 中，部分传递依赖要求 Swift 6.2。消费项目会自行解析依赖，最终工具链要求取决于实际解析结果；请用所选工具链执行 `swift package resolve` 和 `swift build` 验证。

### Swift Package Manager

在你的 `Package.swift` 的 `dependencies` 中添加依赖。以下示例使用本组织 fork 的规范 URL，并固定到已存在的提交：

```swift
.package(
    url: "https://github.com/mcp-everywhere/swift-mcp-ble-transport.git",
    revision: "919dd13407fdf013ae494ba80bd5c04eb406e8e7"
)
```

在使用该库的 target 的 `dependencies` 中加入产品，`package` 使用与 URL 一致的连字符 identity：

```swift
.product(name: "SwiftMCPBLETransport", package: "swift-mcp-ble-transport")
```

同时在消费项目的 `platforms` 中声明适用的最低系统版本，例如 `.iOS(.v16)` 或 `.macOS(.v13)`。上述提交固定了本包源码，其传递依赖仍由消费项目解析；验证后保留消费项目自己的 `Package.resolved`，以便复现依赖版本。

## 快速使用

```swift
import MCP
import SwiftMCPBLETransport

let client = Client(name: "BLEClient", version: "1.0.0")
let transport = BLEMCPTransport(
    options: BLETransportOptions(
        namePrefix: "MCP",
        scanTimeout: .seconds(10)
    )
)
let result = try await client.connect(transport: transport)
let (tools, _) = try await client.listTools()
print(result.capabilities)
print(tools.map(\.name))
```

## 默认 UUID

- Service: `00001999-0000-1000-8000-00805f9b34fb`
- RX: `4963505f-5258-4000-8000-00805f9b34fb`
- TX: `4963505f-5458-4000-8000-00805f9b34fb`

## 协议分片

- 单包：`TYPE_SINGLE`
- 多包首包：`TYPE_START + totalLength(4 bytes, big-endian)`
- 中间包：`TYPE_CONT`
- 结束包：`TYPE_END`
- 序号使用 6-bit 循环计数。

## iOS 权限

需要在应用 `Info.plist` 中添加：

- `NSBluetoothAlwaysUsageDescription`
- `NSBluetoothPeripheralUsageDescription`
