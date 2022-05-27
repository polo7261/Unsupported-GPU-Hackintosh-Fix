# 黑蘋果不支持 AMD 顯卡的修改教學

本指南旨在讓某些 AMD 顯卡在 Mac OS 中工作。目前，得知 XTX varint 的 6900XT 卡在 Mac OS Big Sur 和 Monterey 下運行良好。但是，目前不支持 6900XT 的 XTXH 變體，例如迪蘭恆進 RX 6900XT Ultimate（撰寫本指南時為 Mac OS Monterey 12.0.1）。

如果您也想查看它，我已經發布了我的 RX 6900XT 在另一個 [repository](https://github.com/TylerLyczak/Hackintosh-10850k-ASUS-Z490-XII-Hero-6900XT) 儲存庫。


## Prerequisites
---

* 按照 [OpenCore dortania guide](https://dortania.github.io/OpenCore-Install-Guide/) 為您的系統獲取正確的配置設置
* 獲取最地餓 [Whatevergreen](https://github.com/acidanthera/WhateverGreen) kext 作為對設備 ID GPU 欺騙的支持已在 1.5.2 中添加
* 安裝 [gfxutil](https://github.com/acidanthera/gfxutil)
* 安裝 [IORegistryExplorer](https://github.com/vulgo/IORegistryExplorer)
* 安裝 [MaciASL](https://github.com/acidanthera/MaciASL)
* 安裝 [ProperTree](https://github.com/corpnewt/ProperTree) 或任何程序來編輯 plist 文件


## Getting Info

#### Gfxutil
---

* 運行 `gfxutil` (./gfxutil)
* 查找以 GFX0@0 結尾的條目
    * 例如我的顯卡路徑為： `03:00.0 1002:73bf /PCI0@0/PEG1@1/PEGP@0/BRG0@0/GFX0@0 = PciRoot(0x0)/Pci(0x1,0x0)/Pci(0x0,0x0)/Pci(0x0,0x0)/Pci(0x0,0x0)`
* 牢記 GFX0@0 的路徑
    * Etc. `/PCI0@0/PEG1@1/PEGP@0/BRG0@0/GFX0@0`
* ![GFXUTIL Output](/assets/gfxutil_pic.png)


#### IORegestryExplorer
---

* 運行 `IORegistryExplorer` app
* 在左上角，搜索 GFX0@0
* 這將打開一條通往 GFX0@0 的路徑
    * 如果不支持 gpu，您將看到與 gfxutil 給您的路徑不同的路徑
    * 例如，我的路徑是 `PCI0@0/PEG1@1/PEP@0/pci-device@0/GFX@0`
    * 也寫下這條路徑


## 製作 SSDT-BRG0.aml
---

* 下載 SSDT-BRG0.dsl 你可以從 [OpenCore repository](https://github.com/acidanthera/OpenCorePkg) 取得。
    * 如果下載了 OpenCore 版本，請從以下位置獲取文件 `(OpenCoreFolderPath)/Docs/AcpiSamples/Source` 
    * 如果克隆了 repo，請從 `(OpenCoreRepoPath)/Docs/AcpiSamples/Source`
    * 我在這個 repo 中包含了我自己修改過的這個文件的版本。你也可以嘗試使用它
* 用 MaciASL 打開這個文件
* 在第 12 和 14 行（我的文件為 13 和 16），您將 External 看到 Scope
    * 列出的默認路徑是 `PCI0.PEG0.PEGP`
    * 您需要將其更改為由`gfxutil`
        * 例如，我上面的路徑是`PCI0@0/PEG1@1/PEP@0/pci-device@0/GFX@0`, 所以我會游 `PCI0.PEG0.PEGP` 改成 `PCI0.PEG1.PEGP`
* 在第 20 行（我的文件為 22），有 `Device`.
    * 假設 `gfxutil` path to `GFX0@0` 如果出現如 `BRG0`, 那麼你也必須要跟著修改
* 完成更改後，將文件另存為 `SSDT-BRG0.aml` 文件格式為 `ACPI Machine Language Binary`
* 在您的 EFI/OC/ACPI 文件夾中添加文件


## Edit config.plist
---

* 使用 plist 編輯器程序打開 開啟 config.plist 文件
* 安照路徑 `ACPI -> Add -> (next biggest number)`, 添加 SSDT-BRG0.aml 文件
    * ProperTree 可以使用 Cmd+R 自動添加
    * ![ACPI Section](/assets/acpi_pic.png)
* 在 下DeviceProperties -> Add，使用找到的 gfx 卡的路徑名稱創建一個新子項，gfxutil它是 Dictionary 類型
    * 透過 `gfxutil` 我找例顯卡的路徑
        * 舉例 `PciRoot(0x0)/Pci(0x1,0x0)/Pci(0x0,0x0)/Pci(0x0,0x0)/Pci(0x0,0x0)`
    * Make two children under this new dictionary, `device-id` and `model`
        * `device-id` 將是數據類型
        * `model` 將是字符串類型
    *  `device-id` 設定為 `BF730000`
    *  `model` 寫入顯卡的型號。`Radeon RX 6900 XT (XTXH)`
    * ![DeviceProperties Section](/assets/device_pic.png)
* 接下來 `NVRAM -> Add -> 7C436110-AB2A-4BBB-A880-FE41995C9F82 -> boot-args`, 引導參數，即 append the boot arg `agdpmod=pikera`
    * ![NVRAM Section](/assets/nvram_pic.png)
* 保存文件


## 測試引導文件 config
---

* R重新啟動您的 hackintosh 並查看結果
* 用 gpu 檢查金屬分數，看看它是否工作

## 其他測試
---

* 如果使用 VDADecoder，它可能不會說你有視頻加速
    * 要解決此問題，請在終端中輸入 `defaults write com.apple.AppleGVA gvaForceAMDAVCDecode -boolean yes`


## 反饋
---

* 如果本指南有任何問題，請打開一個問題，我會盡快解決。