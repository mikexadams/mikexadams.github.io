### **Windows Deployment & Maintenance Cheat Sheet**

A structured reference covering key tasks related to Windows imaging, virtual environments, and storage configurations.

## **1\. Creating an Install.wim from a Directory**

To capture an unpacked Windows installation as an install.wim:

```
Dism /Capture-Image /ImageFile:C:\path\to\install.wim /CaptureDir:C:\path\to\unpacked-image /Name:"Windows Image"
```

- `/ImageFile:` specifies the output `.wim`.
    
- `/CaptureDir:` points to the extracted Windows installation folder.
    
- `/Name:` defines the image name.
    

### **Ensuring Integrity & Maximum Compression**

To verify `.wim` integrity:

```
wimlib-imagex optimize install.wim --check
```

For highest compression:

```
wimlib-imagex capture C:\path\to\unpacked-image install.wim --compress=LZX:100
```

## **2\. Handling ISO Mounting**

### **Finding Mounted ISO File Path**

```
Get-Volume <drive letter> | Get-DiskImage | Select-Object ImagePath
```

### **Mounting ISO to an Empty Directory**

```
Mount-DiskImage -ImagePath "C:\path\to\file.iso" -NoDriveLetter -StorageType ISO
diskpart
select volume <volume number>
assign mount="C:\empty-folder"
```

## **3\. Creating a Bootable VHDX with Windows**

### **Steps:**

1. **Create VHDX**:
    
    ```
    New-VHD -Path "C:\VMs\Windows.vhdx" -SizeBytes 30GB -Dynamic
    Mount-VHD "C:\VMs\Windows.vhdx"
    ```
    
2. **Partition the Disk** (`diskpart`):
    
    ```
    select disk 1
    clean
    convert gpt
    create partition efi size=100
    format fs=fat32 quick
    assign letter=S
    create partition msr size=16
    create partition primary
    format fs=ntfs quick label=OS
    assign letter=C
    exit
    ```
    
3. **Apply Windows Image**:
    
    ```
    Dism /Apply-Image /ImageFile:C:\path\to\install.wim /Index:1 /ApplyDir:C:\
    ```
    
4. **Install Boot Files**:
    
    ```
    bcdboot C:\Windows /s S: /f UEFI
    ```
    
5. **Detach & Use in Hyper-V**:
    
    ```
    Dismount-VHD "C:\VMs\Windows.vhdx"
    New-VM -Name "WinVM" -MemoryStartupBytes 4GB -Generation 2 -VHDPath "C:\VMs\Windows.vhdx"
    Start-VM -Name "WinVM"
    ```
    

## **4\. Windows Sandbox vs. Containers**

### **Windows Sandbox**

- **Temporary** isolated Windows instance.
    
- **Resets on close** (not persistent).
    
- **Quick setup** for testing apps.
    

### **Windows Containers**

- **Windows Server Containers** (Docker-based, share host kernel).
    
- **Hyper-V Containers** (isolated Windows kernel).
    
- Run Windows-based images like:
    
    ```
    docker pull mcr.microsoft.com/windows/servercore:ltsc2022
    docker run -it --name mycontainer mcr.microsoft.com/windows/servercore:ltsc2022 cmd
    ```
    

## **5\. File System & Security**

### **Mounting Any Volume to an NTFS Folder**

```
diskpart
select volume <volume number>
assign mount="C:\empty-folder"
```

### **Understanding NTFS DACL Entry**

`Users:(OI)(CI)F` means:

- **Users**: Applies to all users.
    
- **(OI)**: Object Inherit (files inherit permissions).
    
- **(CI)**: Container Inherit (subfolders inherit permissions).
    
- **F**: Full Control (read, modify, delete).
