# Phishing

### Содержание:

  - [Настройка почтового сервера и пользователей](#Почта)
  - [Настройка gophish шаблонов](#Gophish)
  - [Проверка работы](#Проверка)



#### Почта

  В первую очередь настраиваем сертефикат в уже склонированном репозитории. Для запускаем процесс создания сертификата

  ```
  docker run -ti --rm -v "$(pwd)"/config/ssl:/tmp/docker-mailserver/ssl -h mail.domain.com -t tvial/docker-mailserver generate-ssl-certificate
  
  ```

  Затем заполняем все необходимое в сертефикате. Важно, чтобы пароль солдержал цифры, латинские буквы в верхнем и нижнем регистрах, а так же спецсимволы. 

  ВАЖНО! Вводимые поля должны быть идентичны, кроме одного поля Common Name (e.g. server FQDN or YOUR name). Здесь в первым случае надо указать domain.com, во втором mail.domain.com

  ```
		CA certificate filename (or enter to create)

		Making CA certificate ...
		====
		openssl req  -new -keyout ./demoCA/private/cakey.pem -out ./demoCA/careq.pem 
		Generating a RSA private key
		...........................+++++
		.................................................................................................................................+++++
		writing new private key to './demoCA/private/cakey.pem'
		Enter PEM pass phrase:
		Verifying - Enter PEM pass phrase:
		-----
		You are about to be asked to enter information that will be incorporated
		into your certificate request.
		What you are about to enter is what is called a Distinguished Name or a DN.
		There are quite a few fields but you can leave some blank
		For some fields there will be a default value,
		If you enter '.', the field will be left blank.
		-----
		Country Name (2 letter code) [AU]:RU
		State or Province Name (full name) [Some-State]:Moscow
		Locality Name (eg, city) []:Moscow
		Organization Name (eg, company) [Internet Widgits Pty Ltd]:My-company
		Organizational Unit Name (eg, section) []:Red Team      
		Common Name (e.g. server FQDN or YOUR name) []:domain.com
		Email Address []:admin@domain.com

		Please enter the following 'extra' attributes
		to be sent with your certificate request
		A challenge password []:
		An optional company name []:
		==> 0
		====
		====
		openssl ca  -create_serial -out ./demoCA/cacert.pem -days 1095 -batch -keyfile ./demoCA/private/cakey.pem -selfsign -extensions v3_ca  -infiles ./demoCA/careq.pem
		Using configuration from /usr/lib/ssl/openssl.cnf
		Enter pass phrase for ./demoCA/private/cakey.pem:
		Check that the request matches the signature
		Signature ok
		Certificate Details:
		        Serial Number:
		            5d:ac:b5:3f:cc:b6:26:bf:6a:20:dd:c1:ff:08:a0:be:14:1b:e4:3c
		        Validity
		            Not Before: Feb  8 13:40:27 2021 GMT
		            Not After : Feb  8 13:40:27 2024 GMT
		        Subject:
		            countryName               = RU
		            stateOrProvinceName       = Moscow
		            organizationName          = My-company
		            organizationalUnitName    = Red Team
		            commonName                = domain.com
		            emailAddress              = admin@domain.com
		        X509v3 extensions:
		            X509v3 Subject Key Identifier: 
		                E9:4F:D1:CC:9D:09:14:A3:9C:23:68:8E:0E:76:9E:35:AE:22:56:61
		            X509v3 Authority Key Identifier: 
		                keyid:E9:4F:D1:CC:9D:09:14:A3:9C:23:68:8E:0E:76:9E:35:AE:22:56:61

		            X509v3 Basic Constraints: critical
		                CA:TRUE
		Certificate is to be certified until Feb  8 13:40:27 2024 GMT (1095 days)

		Write out database with 1 new entries
		Data Base Updated
		==> 0
		====
		CA certificate is in ./demoCA/cacert.pem
		Ignoring -days; not generating a certificate
		Generating a RSA private key
		..................................................+++++
		..............................................................+++++
		writing new private key to '/tmp/docker-mailserver/ssl/mail.domain.com-key.pem'
		-----
		You are about to be asked to enter information that will be incorporated
		into your certificate request.
		What you are about to enter is what is called a Distinguished Name or a DN.
		There are quite a few fields but you can leave some blank
		For some fields there will be a default value,
		If you enter '.', the field will be left blank.
		-----
		Country Name (2 letter code) [AU]:RU
		State or Province Name (full name) [Some-State]:Moscow
		Locality Name (eg, city) []:Moscow
		Organization Name (eg, company) [Internet Widgits Pty Ltd]:My-company
		Organizational Unit Name (eg, section) []:Red Team       
		Common Name (e.g. server FQDN or YOUR name) []:mail.domain.com
		Email Address []:admin@domain.com

		Please enter the following 'extra' attributes
		to be sent with your certificate request
		A challenge password []:
		An optional company name []:
		Using configuration from /usr/lib/ssl/openssl.cnf
		Enter pass phrase for ./demoCA/private/cakey.pem:
		Check that the request matches the signature
		Signature ok
		Certificate Details:
		        Serial Number:
		            5d:ac:b5:3f:cc:b6:26:bf:6a:20:dd:c1:ff:08:a0:be:14:1b:e4:3d
		        Validity
		            Not Before: Feb  8 13:41:55 2021 GMT
		            Not After : Feb  8 13:41:55 2022 GMT
		        Subject:
		            countryName               = RU
		            stateOrProvinceName       = Moscow
		            organizationName          = My-company
		            organizationalUnitName    = Red Team
		            commonName                = mail.domain.com
		            emailAddress              = admin@domain.com
		        X509v3 extensions:
		            X509v3 Basic Constraints: 
		                CA:FALSE
		            Netscape Comment: 
		                OpenSSL Generated Certificate
		            X509v3 Subject Key Identifier: 
		                2F:6B:63:BE:40:11:03:4E:EC:E7:27:9E:E7:F8:3B:A8:82:9C:84:D9
		            X509v3 Authority Key Identifier: 
		                keyid:E9:4F:D1:CC:9D:09:14:A3:9C:23:68:8E:0E:76:9E:35:AE:22:56:61

		Certificate is to be certified until Feb  8 13:41:55 2022 GMT (365 days)
		Sign the certificate? [y/n]:y


		1 out of 1 certificate requests certified, commit? [y/n]y
		Write out database with 1 new entries
  
  ```
  После этого пересобираем докер контейнеры с новым сертификатом

  ```
  docker-compose up --d
  ```

  Отлично, сертификат готов! Теперь можем переходить к созданию пользователей. Останавливаем наши контейнеры и переходим в деррикторию с почтовым сервером, где и будем создавать пользлвателей

  ```
    # Перейти в деррикторию сервера
    cd MailServer

    # сделать скрипт исполняемым
	chmod +x setup.sh

	# создание email аккаунтов user/password
	./setup.sh -i tvial/docker-mailserver:latest email add admin@domain.com admin123
	./setup.sh -i tvial/docker-mailserver:latest email add user@domain.com user123

	# список email аккаунтов 
	./setup.sh -i tvial/docker-mailserver:latest email list
  ```




#### Gophish






#### Проверка