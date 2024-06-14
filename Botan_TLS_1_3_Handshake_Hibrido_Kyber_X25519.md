# Handshake TLS 1.3 Híbrido: Algoritmo PQC Kyber con curva elíptica x25519

Versión de Botan v3.4.0

Autoría: Julio Cebrián Guirao, [Cipherbit - Grupo Oesía](https://grupooesia.com/cipherbit/).

Este trabajo ha sido realizado como parte de un periodo de prácticas curriculares en Cipherbit, durante la realización de un Grado en Ingeniería Informática en la Universidad de Castilla-La Mancha.

## Índice

- [1. Introducción](#introducción)
- [2. Generación de Certificados](#generación-de-certificados)
    - [2.1. Generación del certificado CA](#generación-del-certificado-ca)
    - [2.2. Generación del certificado del servidor](#generación-del-certificado-del-servidor)
- [3. Scripts de Ejecución](#scripts-de-ejecución)
- [4. Configuración](#configuración)
- [5. Política](#política)
- [6. Prueba](#prueba)
- [7. Analisis de los Resultados](#análisis-de-los-resultados)

## Introducción

El propósito de este documento es definir un ejemplo de la realización de un *handshake* híbrido TLS 1.3 para la librería criptográfica Botan. El proceso de hibridación realizado se compone del algoritmo PQC "Kyber" y de la curva elíptica "x25519". Se indicarán los elementos necesarios para llevar a cabo este proceso, así como los pasos a seguir en cada momento.

Como ya se ha mencionado, la herramienta principal utilizada ha sido la librería criptográfica [Botan](https://github.com/randombit/botan), en su versión 3.4.0. Este [enlace](https://botan.randombit.net/handbook/botan.pdf) contiene la última versión de la guía de uso de la librería. Cabe añadir que el elemento de la librería empleado para llevar a cabo las pruebas es **Botan-CLI** (o Command Line Interface), que permite operar con la librería desde la consola de comandos.

## Generación de Certificados

Para realizar la autenticación en el *handshake* TLS, se ha decidido utilizar un certificado de autenticación, creado a partir de las herramientas de Botan. Este proceso ha sido extraído del siguiente [hilo](https://stackoverflow.com/questions/57542040/how-to-encrypt-data-using-botan-library-and-a-specific-cipher-suite) de información de **StackOverflow**.

Este es un resumen de los pasos a seguir:

### Generación del Certificado CA

    mkdir certdir
    botan keygen > ca_key.pem
    botan gen_self_signed --ca ca_key.pem my_root_authority > certdir/ca_cert.pem

### Generación del Certificado del servidor

    botan keygen > server_key.pem
    botan gen_pkcs10 server_key.pem localhost > server_csr.pem
    botan sign_cert certdir/ca_cert.pem ca_key.pem server_csr.pem > server_cert.pem

Los elementos usados en los scripts de lanzamiento son:
- Para el servidor: **server_cert.pem** y **server_key.pem**
- Para el cliente: **certdir**

Una vez realizado este proceso, el usuario ya cuenta con las herramientas necesarias para el proceso de autenticación básico, por certificados, para Botan-CLI.

## Scripts de Ejecución

Dado que se hace uso de la CLI de la librería, es necesario definir una serie de scripts para ejecutar cada parte involucrada en el protocolo TLS.

En la sección 9.8 de la [guía de uso](https://botan.randombit.net/handbook/botan.pdf) de Botan 3.4.0 se ofrece una breve explicación de los parámetros de servidor y cliente TLS-CLI.

**[Servidor]**

    sudo LD_LIBRARY_PATH=. ./botan tls_server server_cert.pem server_key.pem --port=443 --policy=./politicas_pruebas/pruebas_PQC/politica.rst

**[Cliente]**

    LD_LIBRARY_PATH=. ./botan tls_client localhost --port=443 --trusted-cas=certdir --policy=./politicas_pruebas/pruebas_PQC/politica.rst

Es necesario mencionar que, para la realización de las pruebas, servidor y cliente se ejecutarán desde una misma máquina, en la que el script de lanzamiento de cada uno se ejecutará desde una consola de comandos distinta.

- Los argumentos **./botan tls_server** y **./botan tls_client** denotan el servidor y el cliente Botan-CLI.
- Los elementos del certificado se introducen:
    - En la parte del servidor por medio de los argumentos:
        - **server_cert.pem**
        - **server_key.pem**
    - En la parte del cliente por medio del argumento **trusted-cas**

## Configuración

La configuración usada en este ejemplo está declarada a partir de la codificación paramétrica de Botan:

- La hibridación de KEMs se compone de un esquema PQC y un esquema clásico, correspondientes a:
    - El grupo de intercambio de claves PQC: **Kyber-768-r3**
    - El grupo de intercambio de claves clásico: **x25519**
- El algoritmo de firma es **RSA_PSS_SHA384**.
- El cifrador de flujo usado dentro de la ciphersuite corresponde a **ChaCha20Poly1305**.

Esta configuración debe quedar reflejada en los elementos paramétricos de la política a implementar, mostrados a continuación.

## Política

Declaración de la política:

    allow_tls12 = false
    allow_tls13 = true

    ciphers = ChaCha20Poly1305    

    signature_hashes = SHA-384
    signature_methods = RSA

    key_exchange_groups = x25519/Kyber-768-r3
    key_exchange_groups_to_offer = x25519/Kyber-768-r3

    accepted_client_certificate_types = X509
    accepted_server_certificate_types = X509

Esta configuración debe ir definida dentro de un archivo de texto, cuya dirección debe ser introducida en el argumento **policy** de los [scripts de lanzamiento](#scripts-de-ejecución) de servidor y de cliente, indicando así que corresponde a su política implementada.

## Prueba

En este apartado se muestran los resultados por consola de comandos de los scripts de ejecución en cada una de las partes:

**[Servidor]**

    1.  usuario:~/botan$ sudo LD_LIBRARY_PATH=. ./botan tls_server server_cert.pem server_key.pem --port=443 --policy=./politicas_pruebas/pruebas_PQC/politica.rst
    2.
    3.  Listening for new connections on tcp port 443
    4.
    5.  Handshake complete, TLS v1.3 using CHACHA20_POLY1305_SHA256
    6.
    7.  Session ID 66698394B74413BEA865380D212417FC31E016F973D2B6CA3153EA94C6080E70

**[Cliente]**

    1.  usuario:~/botan$ LD_LIBRARY_PATH=. ./botan tls_client localhost --port=443 --trusted-cas=certdir --policy=./politicas_pruebas/pruebas_PQC/politica.rst
    2.
    3.  Certificate validation status: Verified
    4.
    5.  Handshake complete, TLS v1.3 using CHACHA20_POLY1305_SHA256
    6.  
    7.  Session ID 66698394B74413BEA865380D212417FC31E016F973D2B6CA3153EA94C6080E70
    8.
    9. Handshake complete

## Análisis de los resultados

A modo de resumen, la información significativa que se puede extraer de este resultado es la siguiente:

- Las líneas 5 del servidor y 7 del cliente se observa que:
    - El proceso de handshake se ha completado satisfactoriamente
    - El protocolo seleccionado ha sido TLS 1.3
    - La ciphersuite seleccionada contiene el cifrador de flujo **ChaCha20Poly1305**
    - Las IDs de sesión coinciden, lo que indica que ambas partes se han conectado entre sí.

- La línea 3 del cliente muestra como un certificado ha sido validado, y por ende, que se ha hecho uso de un certificado para el proceso de autenticación.

Cabe mencionar que si un usuario introduce estos mismos parámetros para replicar el experimento, no obtendrá necesariamente el mismo par de IDs de sesión que los que figuran este documento. En cualquier caso, aunque las IDs obtenidas por el usuario sean diferentes a las del documento, estas deben de ser iguales entre sí, para así garantizar la conexión entre su par servidor-cliente.
