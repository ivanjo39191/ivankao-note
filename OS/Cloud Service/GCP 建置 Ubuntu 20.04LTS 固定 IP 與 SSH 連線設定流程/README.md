# GCP 建置 Ubuntu 20.04LTS 固定 IP 與 SSH 連線設定流程

Created: February 1, 2022 5:22 PM
Tags: OS
blog: Done

## GCP 建置 Ubuntu 20.04LTS 固定 IP 與 SSH 連線設定流程

### 申請 GCP 帳號並登入

[Google Cloud Platform (GCP)](https://www.googleadservices.com/pagead/aclk?sa=L&ai=DChcSEwiy4InZmab1AhWbwhYFHU2FB78YABABGgJ0bA&ohost=www.google.com&cid=CAESQOD2rQ2IVSDD7pvatAajNE0FfxrkP0Kk7ridHHUQl3rkGFDt2iYAb7t_EkCkPNUsHbh4G5AMxUym4OE5Iq6P62o&sig=AOD64_0zMM8vhiRGCdwWpBtqzWDnqq8vPQ&q&nis=1&adurl&ved=2ahUKEwibooPZmab1AhVBa94KHWVAA8sQ0Qx6BAgFEAE)

新帳號有90天300美金試用

### 建立虛擬機器

Compute Engine > 建立執行個體

名稱：mygcp

區域：選擇 asia-east1 台灣

機器類型：2vCPU，4 GB 記憶體

開機磁碟：作業系統版本 Ubuntu 20.04LTS，標準永久磁碟 30GB

防火牆：允許 HTTP，HTTPS 流量

建立 (預估每月費用：US$29.52)

### 設定固定 IP

虛擬私有網路 > 外部 IP > 保留靜態位址

區域：選擇 asia-east1 台灣

連接到：mygcp(剛才建立的虛擬機器)

保留

### 防火牆（開放額外 port)

預設已開 80，443

虛擬私有網路 > 防火牆 > 建立防火牆規則

允許目標：網路中所有執行個體（或單一目標）

允許來源：0.0.0.0/24 （請自訂）

通訊協定和埠：tcp： 3000, 8000 （請自訂）

儲存

### ssh 連線

**建立金鑰：**

```bash
# 要發起連線的機器
cd ~/.ssh
ssh-keygen -t rsa -f ~/.ssh/gcp -C gcp_username # 可不設密碼
chmod gcp
cat gcp.pub
ssh-rsa ... gcp_username # 複製整段
```

**GCP 設定：**

Compute Engine > 執行個體 > mygcp(剛才建立的虛擬機器) > 編輯 > SSH 金鑰 > 顯示與編輯

貼上複製的 .pub > 儲存

**連線：**

linux / macos

```bash
# 要發起連線的機器
ssh -i ~/.ssh/gcp_username gcp_username@固定IP
```

vscode-remote 設定

```bash
Host gcp_username
    HostName 固定IP
    User gcp_username
    ServerAliveInterval 60
    IdentityFile ~/.ssh/gcp
```