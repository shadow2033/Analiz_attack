# Атака DCSync

**Цель атаки**: DCSync - получение учетных данных пользователей и администраторов сети, хранимых на контроллерах домена, а также получение доступа к конфиденциальной информации и расширение своих привилегий.

## Описание атаки

DCSync атака направлена на эмуляцию функциональности контроллера домена и выполнение запросов на репликацию данных, как если бы злоумышленник был легитимным DC. Злоумышленник, имея доступ к сети и инструменты, может отправлять запросы DCSync для извлечения учетных данных пользователей, хранящихся на других контроллерах домена.

- Злоумышленник, имея доступ к сети и необходимые инструменты, эмулирует работу контроллера домена.
- Он отправляет запросы DCSync для извлечения учетных данных пользователей, хранящихся на других контроллерах домена.
- Эта атака позволяет злоумышленнику получить хэши паролей пользователей, включая привилегированные учетные записи, такие как администраторы домена.

## Описание правила
Правило "DCSync" предназначено для обнаружения атаки, связанной с получением доступа к данным учетных записей в Active Directory через DCSync.

### Событие ADDS_Replication_With_UserAccount

- **key**: Группирует события по хосту, на котором произошло событие.
- **filter**: Фильтр определяет, какие события будут учтены. В данном случае:
  - `filter::NotFromCorrelator()`: Убедитесь, что событие не было сгенерировано коррелятором.
  - `event_src.title == "windows"`: Проверка, что событие связано с операционной системой Windows.
  - `msgid == "4662"`: Событие с соответствующим идентификатором сообщает о выполнении операции на объекте.
  - `object.type == "19195a5b-6da0-11d0-afd3-00c04fd930c9"`: Этот фильтр ограничивает события на объекты, имеющие класс AD object class.
  - `datafield5 == "0x100"`: Проверяет, что событие связано с привилегированными операциями в Active Directory.
  - `find_substr(datafield2, ...)`: Проверяет, содержит ли datafield2 одно из трех указанных значений, связанных с привилегированными операциями в Active Directory.
  - `find_substr(subject.account.name, "$") == null`: Проверяет, что имя аккаунта субъекта не содержит символ "$".

### Событие ADDS_Replication_With_LocalSystemAccount

- **key**: Группирует события по хосту, на котором произошло событие.
- **filter**: Фильтр определяет, какие события будут учтены. В данном случае:
  - `filter::NotFromCorrelator()`: Убедитесь, что событие не было сгенерировано коррелятором.
  - `event_src.title == "windows"`: Проверка, что событие связано с операционной системой Windows.
  - `msgid == "4662"`: Событие с соответствующим идентификатором сообщает о выполнении операции на объекте.
  - `object.type == "19195a5b-6da0-11d0-afd3-00c04fd930c9"`: Этот фильтр ограничивает события на объекты, имеющие класс AD object class.
  - `datafield5 == "0x100"`: Проверяет, что событие связано с привилегированными операциями в Active Directory.
  - `subject.account.id == "S-1-5-18"`: Этот фильтр проверяет, что субъектом операции является локальная системная учетная запись (LocalSystem) с идентификатором "S-1-5-18".