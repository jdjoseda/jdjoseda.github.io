---
layout: post
title: "Configuración De Correo Saliente Desde Alfresco"
author: Jose David
date: 2014-02-21 16:10:18 +0100
comments: true
categories: Alfresco
---

<div>
<p>Para las notificaciones, invitaciones a sitios, y resto de avisos; Alfresco posee la habilidad de generar y enviar mensajes a un  servidor de correo externo. Para configurar el correo saliente mediante SMTPS, para hacer los envíos desde gmail de forma autenticada y cifrada, simplemente añadiremos un conunto de propiedades al fichero <em>alfresco-global.properties</em>.
<!-- more --></p>


<h2 id="editar-alfresco-global-properties">Editar el fichero alfresco-global.properties</h2>

<p>El fichero lo podemos eencontrar en la siguiente ruta:</P>
{% codeblock %}
Alfresco
|   └── tomcat
|   	  └── shared
|  		    └── classes
|   				   └── alfresco-global.properties
{% endcodeblock %}

<p>Para añadir esta funcionalidad tendremos que abrir el fichero de propiedades con nuestro editor preferido y añadir las siguientes propiedades. </p>

{% codeblock lang:properties alfresco-global.properties %}
### Outbound mail GMAIL ###
	mail.host=smtp.gmail.com
	mail.port=465
	mail.username=usuario@gmail.com
	mail.password=contraseña
	mail.protocol=smtps
	mail.smtps.starttls.enable=true
	mail.smtps.auth=true
{% endcodeblock %}

<div class="alert alert-success">
<p>Para cualquier duda, revisar el siguiente enlace <a href="http://wiki.alfresco.com/wiki/Outbound_E-mail_Configuration">Documentación sobre configuración de correo en Alfresco</a>.</p>
</div>

<h2 id="posibles-errores">Posibles errores que nos podemos encontrar</h2>

<p>Si al probar cualquier opción que requiera el envío de email, como por ejemplo (invitación de un usuario a un sitio); observamos que no se produce el resultado esperado, deberemos abrir el log producido por Alfresco y comprobar si la salida es parecida al código siguiente: </p>

{% codeblock lang:pypylog Alfresco\tomcat\logs\alfrescotomcat-stdout.2014-02-16.log %}
	...
	  154   org.springframework.mail.MailSendException: Mail server connection failed; nested exception is javax.mail.MessagingException: Exception reading response;
	  155    nested exception is:
	  156: 	javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target. Failed messages: javax.mail.MessagingException: Exception reading response;
	  157    nested exception is:
	  158: 	javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target; message exception details (1) are:
	  159  Failed message 1:
	  160  javax.mail.MessagingException: Exception reading response;
	  161    nested exception is:
	  162: 	javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target
	  163  	at com.sun.mail.smtp.SMTPTransport.readServerResponse(SMTPTransport.java:1462)
	  164  	at com.sun.mail.smtp.SMTPTransport.openServer(SMTPTransport.java:1260)
	  ...
	  219  	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
	  220  	at java.lang.Thread.run(Thread.java:724)
	  221: Caused by: javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target
	  222  	at sun.security.ssl.Alerts.getSSLException(Alerts.java:192)
	  223  	at sun.security.ssl.SSLSocketImpl.fatal(SSLSocketImpl.java:1886)
	  ...
	  238  	at com.sun.mail.smtp.SMTPTransport.readServerResponse(SMTPTransport.java:1440)
	  239  	... 57 more
	  240: Caused by: sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target
	  241: 	at sun.security.validator.PKIXValidator.doBuild(PKIXValidator.java:385)
	  242: 	at sun.security.validator.PKIXValidator.engineValidate(PKIXValidator.java:292)
	  243  	at sun.security.validator.Validator.validate(Validator.java:260)
	  244  	at sun.security.ssl.X509TrustManagerImpl.validate(X509TrustManagerImpl.java:326)
	  ...
	  250  	at sun.security.provider.certpath.SunCertPathBuilder.engineBuild(SunCertPathBuilder.java:196)
	  251  	at java.security.cert.CertPathBuilder.build(CertPathBuilder.java:268)
	  252: 	at sun.security.validator.PKIXValidator.doBuild(PKIXValidator.java:380)
	  253  	... 75 more
	  254  2014-02-16 22:57:34,640  WARN  [management.subsystems.ChildApplicationContextFactory] [localhost-startStop-1] Startup of 'email' subsystem, ID: [email, outbound] failed
	...
{% endcodeblock %}

<p>Este error nos dice que no disponemos del certificado necesario para verificar la identidad de los servidores de gmail, para solucionar esto deberemos importar el certificado en el repositorio de llaves de Java.</p>

<h2 id="importar-certificados">Importar el certificado de Gmail desde Windows</h2>

<h4>1.- Descargaremos el certificado del servidor remoto SMTP</h4>
<p>Para ello podremos usar <em>openssl</em> para obtener el certificado, lo descargaremos desde <a href="http://gnuwin32.sourceforge.net/packages/openssl.htm
">openssl for windows</a></p>.
<p>Ejecutamos el siguiente comando</p>
{% codeblock lang:html %}
openssl s_client -connect smtp.gmail.com:465>cert.txt
{% endcodeblock %}
<h4>2.- Abrimos el archivo y copiarremos sólo las líneas entre <code>BEGIN CERTIFICATE</code> y <code>END CERTIFICATE</code> en un archivo llamado gmail.cert</h4>
{% codeblock lang:html %}
-----BEGIN CERTIFICATE-----
MIIDfjCCAuegAwIBAgIDFJoPMA0GCSqGSIb3DQEBBQUAME4xCzAJBgNVBAYTAlVT
MRAwDgYDVQQKEwdFcXVpZmF4MS0wKwYDVQQLEyRFcXVpZmF4IFNlY3VyZSBDZXJ0
aWZpY2F0ZSBBdXRob3JpdHkwHhcNMTAwOTE5MDEzMDUxWhcNMTIxMTIwMTQ0MjI4
WjCB4TEpMCcGA1UEBRMgRFZxc0c5bkIxUGNleTNZVUFzY3otOFNWV0ZnL2Y1aU8x
					    	...
L6AtoCuGKWh0dHA6Ly9jcmwuZ2VvdHJ1c3QuY29tL2NybHMvc2VjdXJlY2EuY3Js
MB0GA1UdDgQWBBRCAb04OmdhLLLRIAtJGNgYnJKQcjANBgkqhkiG9w0BAQUFAAOB
gQCZQVc8nNHGo5Sr1hh9ZMBK2bcivXqLeJkOVt2pQ0OoMWDsq7/ei4njcN5QJXf0
mK3Qb4bUkdJUemS3QITRXVqNnBZaP0XUAKBxK5htwHJLuQ83q71Td6NkqSj4yS35
jM3JXG7LRkr/G6M24RCxBKONckQy+3j1wdy/jZwfisilPg==
-----END CERTIFICATE-----
{% endcodeblock %}

<h4>3.- Importamos el certificado en el repositorio local de llaves de Java</h4>

{% codeblock lang:html %}
keytool -import -alias smtp.gmail.com -keystore "%JAVA_HOME%/jre/lib/security/cacerts" -file gmail.cert
{% endcodeblock %}

<h4>4.- Hay que contestar <code>yes</code> a la pregunta "Trust this certificate? [no]: </h4>
</div>

