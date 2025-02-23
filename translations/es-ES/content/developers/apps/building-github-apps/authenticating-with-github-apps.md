---
title: Autenticarse con GitHub Apps
intro: '{% data reusables.shortdesc.authenticating_with_github_apps %}'
redirect_from:
  - /apps/building-integrations/setting-up-and-registering-github-apps/about-authentication-options-for-github-apps
  - /apps/building-github-apps/authentication-options-for-github-apps
  - /apps/building-github-apps/authenticating-with-github-apps
  - /developers/apps/authenticating-with-github-apps
versions:
  fpt: '*'
  ghes: '*'
  ghae: '*'
  ghec: '*'
topics:
  - GitHub Apps
shortTitle: Autenticación
---


## Generar una llave privada

Después de que creas una GitHub App, necesitarás generar una o más llaves privadas. Utilizarás la llave privada para firmar las solicitudes de token de acceso.

Puedes crear varias llaves privadas y rotarlas para prevenir el tiempo de inactividad si alguna llave se pone en riesgo o se pierde. Para verificar que una llave privada empata con una llave pública, consulta la sección [Verificar llaves privadas](#verifying-private-keys).

Para generar una llave privada:

{% data reusables.user-settings.access_settings %}
{% data reusables.user-settings.developer_settings %}
{% data reusables.user-settings.github_apps %}
{% data reusables.user-settings.modify_github_app %}
5. En "Llaves privadas", da clic en **Generar una llave privada**. ![Generar llave privada](/assets/images/github-apps/github_apps_generate_private_keys.png)
6. Verás una llave privada en formato PEM que se descarga en tu ordenador. Asegúrate de almacenar este archivo, ya que GitHub solo almacena la porción pública de la llave.

{% note %}

**Nota:** Si estás utilizando una biblioteca que requiere de un formato de archivo específico, el archivo PEM que descargues se encontrará en formato `PKCS#1 RSAPrivateKey`.

{% endnote %}

## Verificar las llaves privadas
{% data variables.product.product_name %} genera una huella digital para cada par de llaves pública y privada utilizando la función de hash SHA-256. Puedes verificar que tu llave privada empate con la llave pública almacenada en {% data variables.product.product_name %} generando la huella digital de tu llave privada y comparándola con la huella digital que se muestra en {% data variables.product.product_name %}.

Para verificar una llave privada:

1. Encuentra la huella digital del par de llaves pública y privada que quieras verificar en la sección "Llaves privadas" de tu página de configuración de desarrollador de {% data variables.product.prodname_github_app %}. Para obtener más información, consulta la sección [Generar una llave privada](#generating-a-private-key). ![Huella digital de llave privada](/assets/images/github-apps/github_apps_private_key_fingerprint.png)
2. Genera la huella digital de tu llave privada (PEM) localmente utilizando el siguiente comando:
    ```shell
    $ openssl rsa -in <em>PATH_TO_PEM_FILE</em> -pubout -outform DER | openssl sha256 -binary | openssl base64
    ```
3. Compara los resultados de la huella digital generada localmente con aquella que ves en {% data variables.product.product_name %}.

## Borra las llaves privadas
Puedes eliminar una llave privada que se haya perdido o puesto en riesgo si la borras, pero debes de tener por lo menos una llave privada. Cuando solo tienes una llave, necesitas generar una nueva antes de borrar la anterior. ![Borrar la última llave privada](/assets/images/github-apps/github_apps_delete_key.png)

## Autenticarse como una {% data variables.product.prodname_github_app %}

El autenticarte como una {% data variables.product.prodname_github_app %} te permite hacer un par de cosas:

* Puedes recuperar información administrativa de alto nivel acerca de tu {% data variables.product.prodname_github_app %}.
* Puedes solicitar tokens de acceso para una instalación de la app.

Para autenticarte como una {% data variables.product.prodname_github_app %}, [genera una llave privada](#generating-a-private-key) en formato PEM y descárgala a tu máquina local. Utilizarás esta llave para firmar un [Token Web (JWT) de JSON](https://jwt.io/introduction) y cifrarlo utilizando el algoritmo `RS256`. {% data variables.product.product_name %} revisa que la solicitud se autentique verificando el token con la llave pública almacenada de la app.

Aquí se muestra rápidamente un script de Ruby que puedes utilizar para generar un JWT. Nota que tendrás que ejecutar `gem install jwt` antes de utilizarlo.

<a name="jwt-payload"></a>

```ruby
require 'openssl'
require 'jwt'  # https://rubygems.org/gems/jwt

# Private key contents
private_pem = File.read("YOUR_PATH_TO_PEM")
private_key = OpenSSL::PKey::RSA.new(private_pem)

# Generate the JWT
payload = {
  # issued at time, 60 seconds in the past to allow for clock drift
  iat: Time.now.to_i - 60,
  # JWT expiration time (10 minute maximum)
  exp: Time.now.to_i + (10 * 60),
  # {% data variables.product.prodname_github_app %}'s identifier
  iss: "YOUR_APP_ID"
}

jwt = JWT.encode(payload, private_key, "RS256")
puts jwt
```

`YOUR_PATH_TO_PEM` y `YOUR_APP_ID` son los valores que debes reemplazar. Asegúrate de poner los valores entre comillas dobles.

Utiliza tu identificador de {% data variables.product.prodname_github_app %} (`YOUR_APP_ID`) como el valor para la solicitud del [iss](https://tools.ietf.org/html/rfc7519#section-4.1.1) (emisor) del JWT. Puedes obtener el identificador de {% data variables.product.prodname_github_app %} a través del ping del webhook inicial después de [crear la app](/apps/building-github-apps/creating-a-github-app/), o en cualquier momento desde la página de configuración de la app en la UI de GitHub.com.

Después de crear el JWT, configura el `Header` de la solicitud de la API:

```shell
$ curl -i -H "Authorization: Bearer YOUR_JWT" -H "Accept: application/vnd.github+json" {% data variables.product.api_url_pre %}/app
```

`YOUR_JWT` es el valor que debes reemplazar.

El ejemplo anterior utiliza el tiempo de caducidad máximo de 10 minutos, después del cual, la API comenzará a devolver el error `401`:

```json
{
  "message": "'Expiration' claim ('exp') must be a numeric value representing the future time at which the assertion expires.",
  "documentation_url": "{% data variables.product.doc_url_pre %}"
}
```

Necesitarás crear un nuevo JWT después de que el tiempo caduque.

## Acceder a terminales de API como una {% data variables.product.prodname_github_app %}

Para obtener una lista de las terminales de API de REST que puedes utilizar para obtener información de alto nivel acerca de una {% data variables.product.prodname_github_app %}, consulta la sección "[GitHub Apps](/rest/reference/apps)".

## Autenticarse como una instalación

El autenticarte como una instalación te permite realizar acciones en la API para dicha instalación. Antes de autenticarte como una instalación, debes crear un token de acceso a ésta. Asegúrate de que ya hayas instalado tu GitHub App en por lo menos un repositorio; es imposible crear un token de instalación si una sola instalación. Las {% data variables.product.prodname_github_apps %} utilizan estos tokens de acceso a la instalación para autenticarse. Para obtener más información, consulta la sección "[Instalar las GitHub Apps](/developers/apps/managing-github-apps/installing-github-apps)".

Predeterimenadamente, los tokens de acceso de instalación tienen un alcance de todos los repositorios a los cuales tiene acceso dicha instalación. Puedes limitar el alcance del token de acceso de la instalación a repositorios específicos si utilizas el parámetro `repository_ids`. Consulta la terminal [Crear un token de acceso de instalación para una app](/rest/reference/apps#create-an-installation-access-token-for-an-app) para encontrar más detalles. Los tokens de acceso de instalación cuentan con permisos configurados por la {% data variables.product.prodname_github_app %} y caducan después de una hora.

Para listar las instalaciones para una app autenticada, incluye el JWT [generado anteriormente](#jwt-payload) en el encabezado de autorización en la solicitud de la API:

```shell
$ curl -i -X GET \
-H "Authorization: Bearer YOUR_JWT" \
-H "Accept: application/vnd.github+json" \
{% data variables.product.api_url_pre %}/app/installations
```

La respuesta incluirá una lista de instalaciones en donde cada `id` de instalación pueda utilizarse para crear un token de acceso a la instalación. Para obtener más información acerca del formato de respuesta, consulta la "[Lista de instalaciones para la app autenticada](/rest/reference/apps#list-installations-for-the-authenticated-app)".

Para crear un token de acceso a la instalación, incluye el JWT [que se generó anteriormente](#jwt-payload) en el encabezado de autorización en la solicitud de la API y reemplaza a `:installation_id` con la `id` de la instalación:

```shell
$ curl -i -X POST \
-H "Authorization: Bearer YOUR_JWT" \
-H "Accept: application/vnd.github+json" \
{% data variables.product.api_url_pre %}/app/installations/:installation_id/access_tokens
```

La respuesta incluirá tu token de acceso de instalación, la fecha de caducidad, los permisos del token, y los repositorios a los cuales tiene acceso. Para obtener más información acerca del formato de respuesta, consulta la terminal [Crear un token de acceso de instalación para una app](/rest/reference/apps#create-an-installation-access-token-for-an-app).

Para autenticarte con un token de acceso de instalación, inclúyela en el encabezado de Autorización en la solicitud de la API:

```shell
$ curl -i \
-H "Authorization: Bearer YOUR_INSTALLATION_ACCESS_TOKEN" \
-H "Accept: application/vnd.github+json" \
{% data variables.product.api_url_pre %}/installation/repositories
```

`YOUR_INSTALLATION_ACCESS_TOKEN` es el valor que debes reemplazar.

{% note %}

**Nota:** {% data reusables.getting-started.bearer-vs-token %}

{% endnote %}

## Acceder a las terminales de la API como una instalación

Para encontrar un listado de las terminales de la API de REST disponibles para utilizarse con las {% data variables.product.prodname_github_apps %} utilizando un token de acceso de instalación, consulta la sección "[Terminales Disponibles](/rest/overview/endpoints-available-for-github-apps)".

For a list of endpoints related to installations, see "[Installations](/rest/reference/apps#installations)."

## Acceso a Git basado en HTTP mediante una instalación

Las instalaciones con [permisos](/apps/building-github-apps/setting-permissions-for-github-apps/) en los `contents` de un repositorio pueden utilizar su token de acceso de instalación para autenticarse para acceso a Git. Utiliza el token de acceso de instalación como la contraseña HTTP:

```shell
git clone https://x-access-token:&lt;token&gt;@github.com/owner/repo.git
```
