# Правило Named_Pipe_Impersonation_PrivEsc

"Named Pipe Impersonation Privilege Escalation" - это техника эскалации привилегий, которая использует именованные каналы для получения повышенных привилегий на целевой системе. Атакующий создает именованный канал с помощью инструмента Meterpreter, а затем запускает командный интерпретатор (cmd.exe) в контексте системы (SYSTEM), подключая его к созданному именованному каналу. Это позволяет атакующему выполнять команды с привилегиями SYSTEM и получать полный контроль над целевой системой.

## Методы атаки:

1. **ELEVATE_TECHNIQUE_SERVICE_NAMEDPIPE**: Этот метод использует Meterpreter для создания именованного канала. Затем локально на системе создается командный интерпретатор (cmd.exe), который подключается к именованному каналу Meterpreter. Таким образом, атакующий может олицетворять привилегии системы (SYSTEM) и выполнять команды от ее имени. Важно отметить, что этот метод работает только с родным Windows Meterpreter и требует прав администратора.

2. **ELEVATE_TECHNIQUE_SERVICE_NAMEDPIPE2**: Этот метод аналогичен предыдущему, но вместо создания пользователя SYSTEM, используется файл DLL, записанный на диск, который затем запускается с помощью rundll32.exe в контексте SYSTEM. Файл DLL подключается к Meterpreter, что позволяет атакующему получить привилегии SYSTEM.

3. **ELEVATE_TECHNIQUE_SERVICE_TOKENDUP**: Этот метод отличается от предыдущих. Он предполагает, что у атакующего уже есть дубликат токена SYSTEM. Токены SYSTEM дублируются через все запущенные службы, чтобы найти ту, которая использует SYSTEM. Затем атакующий использует этот токен для запуска службы в памяти с привилегиями SYSTEM и передачи потока от Meterpreter к этой службе. Таким образом, атакующий получает привилегии SYSTEM и может выполнить команды с этими привилегиями.

## Цель атаки:

Целью атаки является получение повышенных привилегий на целевой системе, в частности, привилегий системы. Это позволяет атакующему выполнять команды и операции с высокими привилегиями, что может привести к полному контролю над системой и возможности выполнения различных вредоносных операций.

## Важные замечания:

- Для успешной атаки требуется наличие прав администратора.
- Атакующий должен обладать специализированными инструментами, такими как Meterpreter, для выполнения этой атаки.
- Следует заметить, что эта атака может быть обнаружена и предотвращена путем реализации соответствующих мер безопасности, таких как мониторинг именованных каналов и ограничение доступа к ним.

## Описание правила

Правило "Named_Pipe_Impersonation_PrivEsc" охватывает несколько событий, связанных с созданием и использованием именованных каналов (pipes) в операционных системах Windows.

### Событие Pipe_Created:

- **key:** Формируется на основе хоста (event_src.host) и имени именованного канала (object.name).
- **filter:**
  - `filter::NotFromCorrelator()`: Исключает события, созданные из коррелятора.
  - `event_src.title == "sysmon"`: Фильтрует события, связанные только с Sysmon.
  - `msgid == "17"`: Ожидает события с определенным идентификатором сообщения.
  - `lower(subject.account.name) != "system"`: Исключает события, связанные с учетной записью "system".

### Событие Service_Create:

- **key:** Формируется на основе хоста (event_src.host) и имени службы (object.name).
- **filter:**
  - `filter::NotFromCorrelator()`: Исключает события, созданные из коррелятора.
  - `event_src.title == "windows"`: Фильтрует события, связанные только с Windows.
  - `msgid == "7045"`: Ожидает события с определенным идентификатором сообщения.
  - `regex(lower(object.process.cmdline), "cmd.exe.?>.\pipe|rundll32.exe.?.dll,[a-z]\s/p:[a-z]+", 0) != null`: Использует регулярное выражение для фильтрации событий, связанных с выполнением команд cmd.exe или rundll32.exe с определенными параметрами, что указывает на создание или использование именованных каналов.

### Событие IPC_Access:

- **key:** Формируется на основе хоста (event_src.host) и имени хранилища (object.storage.name).
- **filter:**
  - `filter::NotFromCorrelator()`: Исключает события, созданные из коррелятора.
  - `event_src.title == "windows"`: Фильтрует события, связанные только с Windows.
  - `msgid == "5145"`: Ожидает события с определенным идентификатором сообщения.
  - `in_list(["127.0.0.1", "::1"], src.ip)`: Фильтрует события, связанные с доступом к хранилищу (IPC) только с локальных адресов (127.0.0.1 и ::1).

### Событие Pipe_Connected:

- **key:** Формируется на основе хоста (event_src.host) и имени именованного канала (object.name).
- **filter:**
  - `filter::NotFromCorrelator()`: Исключает события, созданные из коррелятора.
  - `event_src.title == "sysmon"`: Фильтрует события, связанные только с Sysmon.
  - `msgid == "18"`: Ожидает события с определенным идентификатором сообщения.
  - `in_list(["cmd.exe", "rundll32.exe", "system"], lower(subject.process.name))`: Фильтрует события, связанные с определенными процессами (cmd.exe, rundll32.exe, system).
