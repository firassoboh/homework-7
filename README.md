# homework-7


## 1. Broken Authentication via OSINT Password Reset Bypass (Jim's Account)

* **OWASP Top 10 Category:** A07:2021-Identification and Authentication Failures




### Exploitation Steps
Перейдите на страницы товаров и изучите публичные отзывы, чтобы найти адрес электронной почты пользователя: jim@juice-sh.op.
Обратите внимание на подсказки, оставленные этим пользователем в отзывах и полях с адресом, — они активно отсылают к вселенной «Звёздного пути» (например, в отзывах упоминаются «Звёздный флот», «репликаторы», а в адресе доставки указано «USS Enterprise, 1701»).
Сделайте вывод, что профиль пользователя имитирует вымышленного персонажа — Джеймса Т. Кирка.
Перейдите на страницу сброса пароля по адресу http://localhost:3000/#/forgot-password и введите jim@juice-sh.op, чтобы увидеть контрольный вопрос: «Отчество вашего старшего брата?».
Изучив семейную историю персонажа, выясните, что его старшего брата зовут Джордж Сэмюэл Кирк.
Введите Samuel в качестве ответа на контрольный вопрос и задайте новый пароль.

### Resulting Outcome

Приложение проверяет ответ, подтверждает сброс пароля и позволяет злоумышленнику полностью скомпрометировать учётную запись целевого пользователя — без доступа к его исходному паролю или почтовому ящику
----------------------------------------------------------------------------------------------------------
## 2. Vulnerability Name: Authentication Bypass via SQL Injection (Login Bender)

* **OWASP Top 10 Category:** A03:2021-Injection



### Exploitation Steps
Перейдите на страницу входа по адресу http://localhost:3000/#/login
Введите в поле «Email» адрес целевого пользователя вместе с терминатором строки SQL и последовательностью для комментария: bender@juice-sh.op'--
В поле «Password» введите произвольные символы
Нажмите кнопку «Log In», чтобы отправить полезную нагрузку
### Resulting Outcome
Приложение полностью пропускает этап проверки пароля и предоставляет полный доступ к сеансу аутентификации в учётной записи пользователя Бендера — это демонстрирует полный сбой механизмов контроля аутентификации.
-----------------------------------------------------------
3.## 5. Vulnerability Name: Information Disclosure via UNION-Based SQL Injection (Database Schema)
* **Severity:** High
* **OWASP Top 10 Category:** A03:2021-Injection

### Description
The application's product search functionality accepts user input via the `q` query string parameter and uses it dynamically within a backend SQLite execution statement without proper sanitization or parameterization. An attacker can break out of the string context and append a secondary query using the `UNION SELECT` operator. This allows them to read sensitive system configuration tables, such as `sqlite_master`, and extract the complete data structure of the internal database.

### Exploitation Steps
1. Intercept a legitimate search request on the Juice Shop home interface using the Burp Suite Proxy module.
2. Transfer the request context to the Repeater tab for manual manipulation.
3. Modify the top-line query path parameter `q` to inject the breakout sequence and table definition extraction string:
   `GET /rest/products/search?q=banana%27%29+UNION+SELECT+20%2Csql%2C3%2C4%2C5%2C6%2C7%2C8%2C9+FROM+sqlite_master-- HTTP/1.1`
4. Execute the modified request and capture the server's backend schema properties directly from the response panel.

### Resulting Outcome
The application successfully executed the appended query against the internal SQLite metadata structure. As shown in the Burp Repeater analysis below, the server responded with an **HTTP/1.1 200 OK** and populated the `"data"` JSON array with the raw layout text definitions (`CREATE TABLE ...`) for internal objects like the Addresses and Users tables.

![Burp Suite Successful SQL Injection Schema Dump](image_1teMAN.png)



