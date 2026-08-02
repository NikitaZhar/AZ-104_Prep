Tags: #AZ-104

# План нумерации MS Learn (было → стало)

## Принцип

Номер = **трек.модуль.урок** (`P.M.L`). Первая цифра — learning path, вторая — модуль, третья — урок. Вложенность видна на всю глубину прямо в имени файла. Принадлежность к официальному LP — **не в номере**, а тегом `#вне-LP`: тема лежит там, где ей место по смыслу, а флаг «в LP / вне LP» фильтруется отдельно.

Треки:

- **0** — Extra & Tools (вне LP): PowerShell, CLI, портал, ARM/Bicep
- **1** — Prerequisites (Fundamentals) — текущие модули 01–08
- **2** — Manage identities & governance
- **3** — Implement & manage storage
- **4** — Deploy & manage compute
- **5** — Configure & manage virtual networks
- **6** — Monitor & back up

Числа 2–6 зарезервированы под будущие юниты подготовки к экзамену. Savill и верхнеуровневые заметки-индексы (`AZ-104 План подготовки`, `MS Learn…`, `Microsoft Certified…`) не трогаются.

## Целевая структура

```
MS_Learn/
  0 Extra & Tools (вне LP)/
     0.1 PowerShell   0.2 Azure CLI   0.3 The Azure portal   0.4 ARM templates and Bicep files
  1 Prerequisites (Fundamentals)/
     1.1 Describe cloud computing/
        1.1 Describe cloud computing        (индекс модуля)
        1.1.1 … 1.1.7
     1.2 … 1.8 (аналогично)
  2 Manage identities & governance/
     2.1 Manage Microsoft Entra users and groups
  3 Implement & manage storage/        (резерв)
  4 Deploy & manage compute/           (резерв)
  5 Configure & manage virtual networks/ (резерв)
  6 Monitor & back up/                 (резерв)
```

## Объём

Переименовывается **78 файлов** (77 в треке 1 + Extra, 1 из Others → трек 2). Плюс обновление **~216 ссылок** `[[…]]` (в полях `prev`/`next` и оглавлениях модулей) под новые имена. HTML-исходники получают имя модуля (`1.8 … .html`). Заметка `0.1 PowerShell` содержит битую ссылку `[[00.01.01]]` — она сломана уже сейчас, чинить отдельно.

## Таблица переименований

| Было | Стало |
|---|---|
| `00 Not LP Extra Materials/00.01 PowerShell (Extra Material).md` | `0 Extra & Tools (вне LP)/0.1 PowerShell.md` |
| `00 Not LP Extra Materials/00.02 Azure CLI.md` | `0 Extra & Tools (вне LP)/0.2 Azure CLI.md` |
| `00 Not LP Extra Materials/00.03 The Azure portal.md` | `0 Extra & Tools (вне LP)/0.3 The Azure portal.md` |
| `00 Not LP Extra Materials/00.04 ARM templates and Bicep files.md` | `0 Extra & Tools (вне LP)/0.4 ARM templates and Bicep files.md` |
| `01 Describe cloud computing/001_UP_AZ104_Prerequisites_Describe_Cloud_Computing.html` | `1 Prerequisites (Fundamentals)/1.1 Describe cloud computing/1.1 Describe cloud computing.html` |
| `01 Describe cloud computing/01 Describe cloud computing.md` | `1 Prerequisites (Fundamentals)/1.1 Describe cloud computing/1.1 Describe cloud computing.md` |
| `01 Describe cloud computing/01.01 Introduction to Microsoft Azure Fundamentals.md` | `1 Prerequisites (Fundamentals)/1.1 Describe cloud computing/1.1.1 Introduction to Microsoft Azure Fundamentals.md` |
| `01 Describe cloud computing/01.02 Introduction to cloud computing.md` | `1 Prerequisites (Fundamentals)/1.1 Describe cloud computing/1.1.2 Introduction to cloud computing.md` |
| `01 Describe cloud computing/01.03 What is cloud computing.md` | `1 Prerequisites (Fundamentals)/1.1 Describe cloud computing/1.1.3 What is cloud computing.md` |
| `01 Describe cloud computing/01.04 Describe the shared responsibility model.md` | `1 Prerequisites (Fundamentals)/1.1 Describe cloud computing/1.1.4 Describe the shared responsibility model.md` |
| `01 Describe cloud computing/01.05 Define cloud models.md` | `1 Prerequisites (Fundamentals)/1.1 Describe cloud computing/1.1.5 Define cloud models.md` |
| `01 Describe cloud computing/01.06 Describe the consumption-based model.md` | `1 Prerequisites (Fundamentals)/1.1 Describe cloud computing/1.1.6 Describe the consumption-based model.md` |
| `01 Describe cloud computing/01.07 Summary.md` | `1 Prerequisites (Fundamentals)/1.1 Describe cloud computing/1.1.7 Summary.md` |
| `02 Describe the benefits of using cloud services/002_AZ104_Prerequisites_Describe_the_Benefits_of_Using_Cloud_Services.html` | `1 Prerequisites (Fundamentals)/1.2 Describe the benefits of using cloud services/1.2 Describe the benefits of using cloud services.html` |
| `02 Describe the benefits of using cloud services/02 Describe the benefits of using cloud services.md` | `1 Prerequisites (Fundamentals)/1.2 Describe the benefits of using cloud services/1.2 Describe the benefits of using cloud services.md` |
| `02 Describe the benefits of using cloud services/02.01 Introduction.md` | `1 Prerequisites (Fundamentals)/1.2 Describe the benefits of using cloud services/1.2.1 Introduction.md` |
| `02 Describe the benefits of using cloud services/02.02 Describe the benefits of high availability and scalability in the cloud.md` | `1 Prerequisites (Fundamentals)/1.2 Describe the benefits of using cloud services/1.2.2 Describe the benefits of high availability and scalability in the cloud.md` |
| `02 Describe the benefits of using cloud services/02.03 Describe the benefits of reliability and predictability in the cloud.md` | `1 Prerequisites (Fundamentals)/1.2 Describe the benefits of using cloud services/1.2.3 Describe the benefits of reliability and predictability in the cloud.md` |
| `02 Describe the benefits of using cloud services/02.04 Describe the benefits of security and governance in the cloud.md` | `1 Prerequisites (Fundamentals)/1.2 Describe the benefits of using cloud services/1.2.4 Describe the benefits of security and governance in the cloud.md` |
| `02 Describe the benefits of using cloud services/02.05 Describe the benefits of manageability in the cloud.md` | `1 Prerequisites (Fundamentals)/1.2 Describe the benefits of using cloud services/1.2.5 Describe the benefits of manageability in the cloud.md` |
| `02 Describe the benefits of using cloud services/02.06 Describe sustainability considerations in the cloud.md` | `1 Prerequisites (Fundamentals)/1.2 Describe the benefits of using cloud services/1.2.6 Describe sustainability considerations in the cloud.md` |
| `02 Describe the benefits of using cloud services/02.07 Summary.md` | `1 Prerequisites (Fundamentals)/1.2 Describe the benefits of using cloud services/1.2.7 Summary.md` |
| `03 Describe cloud service types/003_AZ104_Prerequisites_Describe_Cloud_Service_Types.html` | `1 Prerequisites (Fundamentals)/1.3 Describe cloud service types/1.3 Describe cloud service types.html` |
| `03 Describe cloud service types/03 Describe cloud service types.md` | `1 Prerequisites (Fundamentals)/1.3 Describe cloud service types/1.3 Describe cloud service types.md` |
| `03 Describe cloud service types/03.01 Introduction.md` | `1 Prerequisites (Fundamentals)/1.3 Describe cloud service types/1.3.1 Introduction.md` |
| `03 Describe cloud service types/03.02 Describe Infrastructure as a Service.md` | `1 Prerequisites (Fundamentals)/1.3 Describe cloud service types/1.3.2 Describe Infrastructure as a Service.md` |
| `03 Describe cloud service types/03.03 Describe Platform as a Service.md` | `1 Prerequisites (Fundamentals)/1.3 Describe cloud service types/1.3.3 Describe Platform as a Service.md` |
| `03 Describe cloud service types/03.04 Describe Software as a Service.md` | `1 Prerequisites (Fundamentals)/1.3 Describe cloud service types/1.3.4 Describe Software as a Service.md` |
| `03 Describe cloud service types/03.05 Summary.md` | `1 Prerequisites (Fundamentals)/1.3 Describe cloud service types/1.3.5 Summary.md` |
| `04 Describe the core architectural components of Azure/004_AZ104_Prerequisites_Describe_Azure_Core_Architectural_Components.html` | `1 Prerequisites (Fundamentals)/1.4 Describe the core architectural components of Azure/1.4 Describe the core architectural components of Azure.html` |
| `04 Describe the core architectural components of Azure/04 Describe the core architectural components of Azure.md` | `1 Prerequisites (Fundamentals)/1.4 Describe the core architectural components of Azure/1.4 Describe the core architectural components of Azure.md` |
| `04 Describe the core architectural components of Azure/04.01 Introduction.md` | `1 Prerequisites (Fundamentals)/1.4 Describe the core architectural components of Azure/1.4.1 Introduction.md` |
| `04 Describe the core architectural components of Azure/04.02 What is Microsoft Azure.md` | `1 Prerequisites (Fundamentals)/1.4 Describe the core architectural components of Azure/1.4.2 What is Microsoft Azure.md` |
| `04 Describe the core architectural components of Azure/04.03 Get started with Azure accounts.md` | `1 Prerequisites (Fundamentals)/1.4 Describe the core architectural components of Azure/1.4.3 Get started with Azure accounts.md` |
| `04 Describe the core architectural components of Azure/04.04 Describe Azure physical infrastructure.md` | `1 Prerequisites (Fundamentals)/1.4 Describe the core architectural components of Azure/1.4.4 Describe Azure physical infrastructure.md` |
| `04 Describe the core architectural components of Azure/04.05 Describe Azure management infrastructure.md` | `1 Prerequisites (Fundamentals)/1.4 Describe the core architectural components of Azure/1.4.5 Describe Azure management infrastructure.md` |
| `04 Describe the core architectural components of Azure/04.06 Summary.md` | `1 Prerequisites (Fundamentals)/1.4 Describe the core architectural components of Azure/1.4.6 Summary.md` |
| `05 Describe Azure compute services/005_AZ104_Prerequisites_Describe_Azure_Compute_Services.html` | `1 Prerequisites (Fundamentals)/1.5 Describe Azure compute services/1.5 Describe Azure compute services.html` |
| `05 Describe Azure compute services/05 Describe Azure compute services.md` | `1 Prerequisites (Fundamentals)/1.5 Describe Azure compute services/1.5 Describe Azure compute services.md` |
| `05 Describe Azure compute services/05.01 Introduction.md` | `1 Prerequisites (Fundamentals)/1.5 Describe Azure compute services/1.5.1 Introduction.md` |
| `05 Describe Azure compute services/05.02 Describe Azure virtual machines.md` | `1 Prerequisites (Fundamentals)/1.5 Describe Azure compute services/1.5.2 Describe Azure virtual machines.md` |
| `05 Describe Azure compute services/05.03 Describe Azure virtual desktop.md` | `1 Prerequisites (Fundamentals)/1.5 Describe Azure compute services/1.5.3 Describe Azure virtual desktop.md` |
| `05 Describe Azure compute services/05.04 Describe Azure containers.md` | `1 Prerequisites (Fundamentals)/1.5 Describe Azure compute services/1.5.4 Describe Azure containers.md` |
| `05 Describe Azure compute services/05.05 Describe Azure functions.md` | `1 Prerequisites (Fundamentals)/1.5 Describe Azure compute services/1.5.5 Describe Azure functions.md` |
| `05 Describe Azure compute services/05.06 Describe AI, machine learning, and IoT Edge services in Azure.md` | `1 Prerequisites (Fundamentals)/1.5 Describe Azure compute services/1.5.6 Describe AI, machine learning, and IoT Edge services in Azure.md` |
| `05 Describe Azure compute services/05.07 Describe application hosting options.md` | `1 Prerequisites (Fundamentals)/1.5 Describe Azure compute services/1.5.7 Describe application hosting options.md` |
| `05 Describe Azure compute services/05.08 Summary.md` | `1 Prerequisites (Fundamentals)/1.5 Describe Azure compute services/1.5.8 Summary.md` |
| `06 Describe Azure networking services/006_UP_AZ104_Prerequisites_Describe_Azure_Networking_Services.html` | `1 Prerequisites (Fundamentals)/1.6 Describe Azure networking services/1.6 Describe Azure networking services.html` |
| `06 Describe Azure networking services/06 Describe Azure networking services.md` | `1 Prerequisites (Fundamentals)/1.6 Describe Azure networking services/1.6 Describe Azure networking services.md` |
| `06 Describe Azure networking services/06.01 Introduction.md` | `1 Prerequisites (Fundamentals)/1.6 Describe Azure networking services/1.6.1 Introduction.md` |
| `06 Describe Azure networking services/06.02 Describe Azure virtual networking.md` | `1 Prerequisites (Fundamentals)/1.6 Describe Azure networking services/1.6.2 Describe Azure virtual networking.md` |
| `06 Describe Azure networking services/06.03 Describe Azure virtual private networks.md` | `1 Prerequisites (Fundamentals)/1.6 Describe Azure networking services/1.6.3 Describe Azure virtual private networks.md` |
| `06 Describe Azure networking services/06.04 Describe Azure ExpressRoute.md` | `1 Prerequisites (Fundamentals)/1.6 Describe Azure networking services/1.6.4 Describe Azure ExpressRoute.md` |
| `06 Describe Azure networking services/06.05 Describe Azure DNS.md` | `1 Prerequisites (Fundamentals)/1.6 Describe Azure networking services/1.6.5 Describe Azure DNS.md` |
| `06 Describe Azure networking services/06.06 Summary.md` | `1 Prerequisites (Fundamentals)/1.6 Describe Azure networking services/1.6.6 Summary.md` |
| `07 Describe Azure storage services/007_UP_AZ104_Prerequisites_Describe_Azure_Storage_Services.html` | `1 Prerequisites (Fundamentals)/1.7 Describe Azure storage services/1.7 Describe Azure storage services.html` |
| `07 Describe Azure storage services/07 Describe Azure storage services.md` | `1 Prerequisites (Fundamentals)/1.7 Describe Azure storage services/1.7 Describe Azure storage services.md` |
| `07 Describe Azure storage services/07.01 Introduction.md` | `1 Prerequisites (Fundamentals)/1.7 Describe Azure storage services/1.7.1 Introduction.md` |
| `07 Describe Azure storage services/07.02 Describe Azure storage accounts.md` | `1 Prerequisites (Fundamentals)/1.7 Describe Azure storage services/1.7.2 Describe Azure storage accounts.md` |
| `07 Describe Azure storage services/07.03 Describe Azure storage redundancy.md` | `1 Prerequisites (Fundamentals)/1.7 Describe Azure storage services/1.7.3 Describe Azure storage redundancy.md` |
| `07 Describe Azure storage services/07.04 Describe Azure storage services.md` | `1 Prerequisites (Fundamentals)/1.7 Describe Azure storage services/1.7.4 Describe Azure storage services.md` |
| `07 Describe Azure storage services/07.05 Identify Azure data migration options.md` | `1 Prerequisites (Fundamentals)/1.7 Describe Azure storage services/1.7.5 Identify Azure data migration options.md` |
| `07 Describe Azure storage services/07.06 Identify Azure file movement options.md` | `1 Prerequisites (Fundamentals)/1.7 Describe Azure storage services/1.7.6 Identify Azure file movement options.md` |
| `07 Describe Azure storage services/07.07 Summary.md` | `1 Prerequisites (Fundamentals)/1.7 Describe Azure storage services/1.7.7 Summary.md` |
| `08 Describe Azure identity, access, and security/008_UP_AZ104_Prerequisites_Describe_Azure_Identity_Access_Security.html` | `1 Prerequisites (Fundamentals)/1.8 Describe Azure identity, access, and security/1.8 Describe Azure identity, access, and security.html` |
| `08 Describe Azure identity, access, and security/08 Describe Azure identity, access, and security.md` | `1 Prerequisites (Fundamentals)/1.8 Describe Azure identity, access, and security/1.8 Describe Azure identity, access, and security.md` |
| `08 Describe Azure identity, access, and security/08.01 Introduction.md` | `1 Prerequisites (Fundamentals)/1.8 Describe Azure identity, access, and security/1.8.1 Introduction.md` |
| `08 Describe Azure identity, access, and security/08.10 Describe Microsoft Defender for Cloud.md` | `1 Prerequisites (Fundamentals)/1.8 Describe Azure identity, access, and security/1.8.10 Describe Microsoft Defender for Cloud.md` |
| `08 Describe Azure identity, access, and security/08.11 Summary.md` | `1 Prerequisites (Fundamentals)/1.8 Describe Azure identity, access, and security/1.8.11 Summary.md` |
| `08 Describe Azure identity, access, and security/08.02 Describe Azure directory services.md` | `1 Prerequisites (Fundamentals)/1.8 Describe Azure identity, access, and security/1.8.2 Describe Azure directory services.md` |
| `08 Describe Azure identity, access, and security/08.03 Describe Azure authentication methods.md` | `1 Prerequisites (Fundamentals)/1.8 Describe Azure identity, access, and security/1.8.3 Describe Azure authentication methods.md` |
| `08 Describe Azure identity, access, and security/08.04 Describe Azure external identities.md` | `1 Prerequisites (Fundamentals)/1.8 Describe Azure identity, access, and security/1.8.4 Describe Azure external identities.md` |
| `08 Describe Azure identity, access, and security/08.05 Describe Azure conditional access.md` | `1 Prerequisites (Fundamentals)/1.8 Describe Azure identity, access, and security/1.8.5 Describe Azure conditional access.md` |
| `08 Describe Azure identity, access, and security/08.06 Describe Azure role-based access control.md` | `1 Prerequisites (Fundamentals)/1.8 Describe Azure identity, access, and security/1.8.6 Describe Azure role-based access control.md` |
| `08 Describe Azure identity, access, and security/08.07 Describe Zero Trust model.md` | `1 Prerequisites (Fundamentals)/1.8 Describe Azure identity, access, and security/1.8.7 Describe Zero Trust model.md` |
| `08 Describe Azure identity, access, and security/08.08 Describe defense-in-depth.md` | `1 Prerequisites (Fundamentals)/1.8 Describe Azure identity, access, and security/1.8.8 Describe defense-in-depth.md` |
| `08 Describe Azure identity, access, and security/08.09 Describe encryption and key management in Azure.md` | `1 Prerequisites (Fundamentals)/1.8 Describe Azure identity, access, and security/1.8.9 Describe encryption and key management in Azure.md` |
| `Others/AZ-104 Manage Entra users and groups.md` | `2 Manage identities & governance/2.1 Manage Microsoft Entra users and groups.md` |

