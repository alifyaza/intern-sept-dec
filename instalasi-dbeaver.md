# Installing DBeaver 24.2.0 on Windows 10

## Introduction
DBeaver is a free, multi-platform database tool that supports any database having a JDBC driver. It provides an intuitive interface for managing databases, running queries, and performing various database operations.

## Prerequisites
- A Windows 10 operating system
- Internet connection
- Administrative privileges on the system

## Steps to Install DBeaver

### 1. Download DBeaver
1. Open your web browser and go to the [DBeaver official website](https://dbeaver.io/download/).
2. Click on the "[Download](https://dbeaver.io/download/)" button.
3. Choose the Windows version and click on the "[Windows (Installer)](https://dbeaver.io/files/dbeaver-ce-latest-x86_64-setup.exe)" link to get the installer.

### 2. Install DBeaver
1. Once the download is complete, navigate to the downloaded file (usually in the `Downloads` folder).
2. Double-click the installer file (`dbeaver-ce-24.2.0-x86_64-setup.exe`).
3. If prompted by the User Account Control, click "Yes" to allow the installer to run.
4. Follow the on-screen instructions:
   - Select the installation location or leave it as default.
   - Choose the components to install or leave the default selections.
   - Click "Next" and then "Install" to begin the installation process.
5. Once the installation is complete, click "Finish" to exit the installer.

### 3. Launch DBeaver
1. After installation, you can launch DBeaver from the Start Menu or by double-clicking the DBeaver icon on your desktop.
2. On the first launch, you may be prompted to import settings from a previous installation. Choose the appropriate option and proceed.

## How to Create a Database Connection

### 1. Open DBeaver
1. Launch DBeaver from the Start Menu or by double-clicking the DBeaver icon on your desktop.

### 2. Create a New Connection
1. Click on the "**New Database Connection**" button in the toolbar or go to `Database > New Database Connection`.
2. In the "Connect to a database" dialog, select the database type you want to connect to (e.g., MySQL, PostgreSQL, Oracle, etc.).
3. Click "Next".

### 3. Configure Connection Settings
1. Enter the connection details:
   For example, the credentials is `maindata/dataset@scan.primo.company.id:1521/MIMO`
   - **Host**: The hostname or IP address of your database server. (Ex: `scan.primo.company.id`)
   - **Port**: The port number on which your database server is running. (Ex: `1521`)
   - **Database**: The name of the database you want to connect to. (Ex: `MIMO`)
   - **Username**: Your database username. (Ex: `maindata`)
   - **Password**: Your database password. (Ex: `dataset`)
2. Optionally, you can test the connection by clicking the "**Test Connection**" button to ensure that the details are correct.
3. Click "**Finish**" to create the connection.

### 4. Connect to the Database
1. Once the connection is created, it will appear in the Database Navigator panel.
2. Double-click the connection to connect to the database.
3. You can now start running queries and managing your database.

## Conclusion
You have successfully installed DBeaver on your Windows 10 machine. You can now start connecting to your databases and managing them efficiently. For more information on how to use DBeaver, refer to the [official documentation](https://dbeaver.io/documentation/).