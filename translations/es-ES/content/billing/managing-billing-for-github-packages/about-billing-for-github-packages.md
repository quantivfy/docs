---
title: Acerca de la facturación para GitHub Packages
intro: 'Si quieres utilizar {% data variables.product.prodname_registry %} con más almacenamiento o transferencia de datos de los que se incluyen en tu cuenta, se te cobrará por este uso adicional.'
product: '{% data reusables.gated-features.packages %}'
redirect_from:
  - /github/setting-up-and-managing-billing-and-payments-on-github/about-billing-for-github-packages
  - /github/setting-up-and-managing-billing-and-payments-on-github/managing-billing-for-github-packages/about-billing-for-github-packages
versions:
  fpt: '*'
  ghec: '*'
type: overview
topics:
  - Packages
  - Spending limits
shortTitle: Acerca de la facturación
---

## Acerca de la facturación para {% data variables.product.prodname_registry %}

{% data reusables.package_registry.packages-billing %}

{% data reusables.package_registry.packages-spending-limit-brief %} Para obtener más información, consulta la sección "[Acerca de los límites de gasto](#about-spending-limits)".

{% note %}

**Actualización de facturación para el almacenamiento de imágenes de contenedor:** Se ha extendido el periodo de uso gratuito para el ancho de banda y almacenamiento de imágenes de contenedor para el {% data variables.product.prodname_container_registry %}. Si estás utilizando el {% data variables.product.prodname_container_registry %}, se te informará por lo menos con un mes de anticipación sobre el inicio de la facturación y se te dará un estimado de cuánto es lo que debes pagar. Para obtener más información acerca del {% data variables.product.prodname_container_registry %}, consulta la sección "[Trabajar con el registro de contenedores](/packages/working-with-a-github-packages-registry/working-with-the-container-registry)".

{% endnote %}

{% ifversion ghec %}
Si compraste {% data variables.product.prodname_enterprise %} mediante un Acuerdo de Microsoft Enterprise, puedes conectar tu ID de Suscripción de Azure a tu cuenta empresarial para habilitar y pagar por el uso de {% data variables.product.prodname_registry %} más allá de las cantidades que se incluyen en tu cuenta. Para obtener más información, consulta la sección "[Conectar una suscripción de Azure a tu empresa](/billing/managing-billing-for-your-github-account/connecting-an-azure-subscription-to-your-enterprise)".
{% endif %}

La transferencia de datos se restablece cada mes, pero no así el uso de almacenamiento.

| Producto                                                              | Almacenamiento | Transferencia de datos (por mes) |
| --------------------------------------------------------------------- | -------------- | -------------------------------- |
| {% data variables.product.prodname_free_user %}                     | 500MB          | 1GB                              |
| {% data variables.product.prodname_pro %}                             | 2GB            | 10GB                             |
| {% data variables.product.prodname_free_team %} para organizaciones | 500MB          | 1GB                              |
| {% data variables.product.prodname_team %}                            | 2GB            | 10GB                             |
| {% data variables.product.prodname_ghe_cloud %}                     | 50GB           | 100GB                            |

Todos los datos de transferencia saliente, cuando se desencadenan mediante {% data variables.product.prodname_actions %}, y aquellos de transferencia entrante desde cualquier origen, son gratuitos. Determinamos que estás descargando paquetes mediante el uso de {% data variables.product.prodname_actions %} cuando ingresas en {% data variables.product.prodname_registry %} utilizando un `GITHUB_TOKEN`.

|                                               | Hospedado | Auto-Hospedado |
| --------------------------------------------- | --------- | -------------- |
| Acceso utilizando un `GITHUB_TOKEN`           | Gratis    | Gratis         |
| Acceso utilizando un token de acceso personal | Gratis    | $              |

El uso de almacenamiento se comparte con los artefactos de compilación que produce {% data variables.product.prodname_actions %} para los repositorios que pertenecen a tu cuenta. Para obtener más información, consulta "[Acerca de la facturación para {% data variables.product.prodname_actions %}](/billing/managing-billing-for-github-actions/about-billing-for-github-actions)".

{% data variables.product.prodname_dotcom %} cobra el uso a la cuenta a la que pertenece el repositorio en donde se publica el paquete. Si tu uso de cuenta sobrepasa estos límites y configuraste un límite de gastos mayor a $0 USD, pagarás $0.008 USD por GB de almacenamiento por día y $0.50 USD por GB de transferencia de datos.

Por ejemplo, si tu organización utiliza {% data variables.product.prodname_team %}, permite los gastos ilimitados, utiliza 150GB de almacenamiento, y tiene 50GB de transferencia de datos durante un mes, ésta tendrá un excedente de 148GB en el almacenamiento y de 40GB en transferencia de datos para ese mes. El almacenamiento excedente costaría $0.008 USD por GB por día o, aproximadamente, $37 USD por un mes de 31 días. El excedente para transferencia de datos costaría $0.50 USD por GB, o $20 USD.

{% data reusables.dotcom_billing.pricing_calculator.pricing_cal_packages %}

Al final del mes, {% data variables.product.prodname_dotcom %} redondea tu transferencia de datos al número de GB más cercano.

{% data variables.product.prodname_dotcom %} calcula tu uso de almacenamiento para cada mes basándose en el uso por hora durante el mismo mes. Por ejemplo, si utilizas 3 GB de almacenamiento durante 10 días de Marzo y 12 GB durante 21 días de Marzo, tu uso de almacenamiento sería:

- 3 GB x 10 días x (24 horas por día) = 720 GB-Horas
- 12 GB x 21 días x (24 horas por día) = 6,048 GB-Horas
- 720 GB-Horas + 6,048 GB-Horas = 6,768 GB-Horas
- 6,768 GB-Horas/ (744 horas por mes) = 9.0967 GB-Meses

Al final del mes, {% data variables.product.prodname_dotcom %} redondea tu almacenamiento al número de MB más cercano. Por lo tanto, tu uso de almacenamiento para marzo sería de 9.097 GB.

Tu uso de {% data variables.product.prodname_registry %} comparte la fecha de facturación, método de pago y recibo existente en tu cuenta. {% data reusables.dotcom_billing.view-all-subscriptions %}

{% data reusables.user-settings.context_switcher %}

## Acerca de los límites de gasto

{% data reusables.package_registry.packages-spending-limit-detailed %}

Para obtener más información sobre cómo administrar y cambiar el límite de gastos de tu cuenta, consulta la sección "[Administrar tu límite de gastos para {% data variables.product.prodname_registry %}](/billing/managing-billing-for-github-packages/managing-your-spending-limit-for-github-packages)".

{% data reusables.dotcom_billing.actions-packages-unpaid-account %}
