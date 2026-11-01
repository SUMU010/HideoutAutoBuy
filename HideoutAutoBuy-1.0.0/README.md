# HideoutAutoBuy —— 藏身处一键购买建造 mod

在建造/升级藏身处建筑时，**一键（F9）自动从跳蚤市场购买缺失的材料并开始建造**。

- 适用版本：**SPT 4.1.5**（本 mod 针对该版本编译，其他版本需重新编译并验证类名）
- 组成：服务器 mod（查价路由）+ 客户端 BepInEx 插件（触发购买与建造）

---

## 一、安装

1. 确认已编译（`build\` 目录下有 `HideoutAutoBuy.dll` 与 `HideoutAutoBuy.Client.dll`）。
2. 在 PowerShell 中运行：

```powershell
Set-ExecutionPolicy -Scope Process Bypass -Force
& "D:\DS_of_EFT\install-hideout-autobuy.ps1"
```

或手动复制两个文件：

| 文件 | 目标位置 |
|---|---|
| `build\HideoutAutoBuy.dll` | `D:\EFT\SPT_Runtime\user\mods\HideoutAutoBuy\HideoutAutoBuy.dll` |
| `build\HideoutAutoBuy.Client.dll` | `D:\EFT\BepInEx\plugins\HideoutAutoBuy.Client.dll` |

3. 重启 SPT 服务器与游戏客户端。

---

## 二、使用方法

1. 进入**藏身处**；
2. **点击选中**一个需要建造/升级的区域（打开它的建造面板即可）；
3. 按下 **F9**；
4. 插件自动完成：
   - 计算该等级缺少的材料（只处理可在跳蚤市场交易的部分）；
   - 查询跳蚤市场**最低价**订单（服务器 mod 报价，排除自己挂的订单、锁定订单和已达购买上限的订单）；
   - 若现金不足或部分材料无货 → 弹出通知并中止（不会乱买）；
   - 用游戏**原生**购买请求逐笔下单（仓库即时更新、手续费照常）；
   - 材料齐后自动触发游戏原生的建造/升级流程。
5. 屏幕通知会提示每一步的结果（成功 / 缺钱 / 无货 / 失败原因）。

> 若区域内已放好部分材料，插件只购买缺口数量。
> 若材料已齐全，按 F9 会直接开始建造（等同点击建造按钮）。
> 现金不足时屏幕通知显示"缺少货币!"（详细差额见 BepInEx 日志）。

---

## 二-补充、BepInEx 配置面板

游戏中按 **F12** 打开 BepInEx Configuration Manager，找到 **HideoutAutoBuy-Client**，可实时调整：

| 配置项 | 默认 | 说明 |
|---|---|---|
| 一键购买快捷键 | F9 | 点击后可重新绑定任意键 |
| 材料集齐自动开始建造 | 开 | 关闭后只买材料、不自动开工 |
| 显示屏幕通知 | 开 | 关闭后结果只写入日志 |
| 自动购买金额上限 | 0（不限） | 单次购买花费超过该值则取消 |

配置文件位置：`BepInEx\config\com.ds.hideoutautobuy.client.cfg`

---

## 三、工作原理（简）

- **客户端插件**（`BepInEx/plugins/HideoutAutoBuy.Client.dll`）
  - 监听 F9；通过 `HideoutScreenOverlay._selectedData` 拿到当前选中区域；
  - 遍历 `Stage.Requirements` 中未满足的 `ItemRequirement`（读取 `IntCount - UserItemsCount` 得到缺口）；
  - 调用服务器报价路由 → 把报价转换为 `CommoditiesToPurchase` → 调用游戏原生
    `EftClientBackendSession.RagfairPurchaseOffers()` 购买（响应走游戏自带管线，仓库自动同步）；
  - 最后调用原生 `AreaData.UpgradeAction()` 开始建造。
- **服务器 mod**（`SPT_Runtime/user/mods/HideoutAutoBuy/`）
  - 注册自定义路由 `POST /client/hideout/autobuy/quote`；
  - 通过 `RagfairOfferHolder.GetOffersByTemplate()` 拉取订单，过滤（自己挂的、锁定的、
    购买限制已满的）后按 `SummaryCost` 升序拆分数量，返回购买计划与预估总价；
  - 同时返回玩家持有卢布数，供客户端判断资金是否充足。

设计原则：**只读不写** —— 服务器 mod 不修改存档，一切购买与建造都走游戏原生请求，风险最小。

---

## 四、修改快捷键

编辑 `HideoutAutoBuy\Client\Plugin.cs`，把 `KeyCode.F9` 换成其它键
（如 `KeyCode.F8`），然后重新编译客户端：

```powershell
Set-ExecutionPolicy -Scope Process Bypass -Force
& "D:\DS_of_EFT\build-hideout-autobuy.ps1" -Client
```

---

## 五、重新编译

无需安装 .NET SDK，编译脚本使用系统自带的 C# 编译器（csc.exe）+ 游戏目录中的程序集：

```powershell
Set-ExecutionPolicy -Scope Process Bypass -Force
& "D:\DS_of_EFT\build-hideout-autobuy.ps1" -Server -Client
```

脚本说明：
- **服务器端**：以 `net10.0` 目标编译（引用 `C:\Program Files\dotnet\shared\Microsoft.NETCore.App\10.0.11`
  与 `D:\EFT\SPT_Runtime` 下的 SPTarkov 程序集）。由于旧 csc 不支持 C# 9 的 `init` 访问器，
  脚本会用 Mono.Cecil 给 `ModMetadata` 补上 `IModMetadata` 接口实现与
  `void modreq(IsExternalInit)` 的 setter 签名。
- **客户端**：以 Unity mono 目标编译（引用游戏 `Managed` 目录 + BepInEx + spt-common），
  编译后自动归一化 `mscorlib 2.0/4.0` 双引用并校验悬空 TypeRef。

---

## 六、常见问题

| 现象 | 处理 |
|---|---|
| 按 F9 无反应 | 先点击选中区域；确认已进入藏身处界面；确认客户端插件已安装（进游戏后日志有 `HideoutAutoBuy client plugin loaded`） |
| 提示"服务器无响应" | 确认服务器 mod 已放入 `SPT_Runtime\user\mods\HideoutAutoBuy\`，重启了 SPT 服务器 |
| 提示"无法在跳蚤市场购买" | 该材料确实无人挂单（或不可交易）；手动补齐后再按 F9 |
| 提示"现金不足" | 会显示预计总价与持有卢布，先卖点东西 |
| 材料买到了但建造没开始 | 再按一次 F9（此时材料已齐，会直接触发建造） |
| 游戏更新 / SPT 版本变化 | 客户端插件的字段/类名可能失效，需重新编译；服务器 mod 需重新对照 API 编译 |

## 七、文件清单

```
D:\DS_of_EFT\
├─ HideoutAutoBuy\
│  ├─ Server\  ModMetadata.cs / HideoutAutoBuyMod.cs / QuoteRouter.cs / QuoteCallback.cs / Models.cs
│  └─ Client\  Plugin.cs / AutoBuyFlow.cs / Dto.cs
├─ build\                         编译产物
│  ├─ HideoutAutoBuy.dll          服务器 mod
│  └─ HideoutAutoBuy.Client.dll   客户端插件
├─ package\HideoutAutoBuy\        按游戏目录结构打包的安装包
├─ build-hideout-autobuy.ps1      编译脚本
└─ install-hideout-autobuy.ps1    安装脚本
```
