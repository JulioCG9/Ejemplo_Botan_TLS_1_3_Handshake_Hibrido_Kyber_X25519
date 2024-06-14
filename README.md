# README

Autoría: Julio Cebrián Guirao, [Cipherbit - Grupo Oesía](https://grupooesia.com/cipherbit/).

Este trabajo ha sido realizado como parte de un periodo de prácticas curriculares en Cipherbit, durante la realización de un Grado en Ingeniería Informática en la Universidad de Castilla-La Mancha.

## Resumen General

Los contenidos del documento [Botan_TLS_1_3_Handshake_Hibrido_Kyber_X25519.md](https://github.com/JulioCG9/Ejemplo_Botan_TLS_1_3_Handshake_Hibrido_Kyber_X25519/blob/main/Botan_TLS_1_3_Handshake_Hibrido_Kyber_X25519.md) tienen como objetivo ejemplificar de forma concisa, directa y gradual los pasos a seguir para realizar un *handshake* híbrido TLS 1.3 por medio de la librería criptográfica [Botan](https://github.com/randombit/botan). Los elementos que componen el proceso de hibridación son el esquema KEM PQC "Kyber" y el esquema KEM clásico de curva elíptica "x25519".

- La versión de Botan empleada ha sido **Botan 3.4.0**.
- La herramienta de Botan con la que se ha establecido el *setup* TLS 1.3 ha sido **Botan-CLI** (*Command Line Interface* de Botan).

## Licencia de uso

Shield: [![CC BY-NC-SA 4.0][cc-by-nc-sa-shield]][cc-by-nc-sa]

Esta obra está bajo una
[Licencia Creative Commons Atribución-NoComercial-CompartirIgual 4.0 Internacional][cc-by-nc-sa].

[![CC BY-NC-SA 4.0][cc-by-nc-sa-image]][cc-by-nc-sa]

[cc-by-nc-sa]: https://creativecommons.org/licenses/by-nc-sa/4.0/deed.es
[cc-by-nc-sa-image]: https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png
[cc-by-nc-sa-shield]: https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg
