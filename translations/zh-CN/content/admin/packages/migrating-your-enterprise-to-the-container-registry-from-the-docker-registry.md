---
title: Migrating your enterprise to the Container registry from the Docker registry
intro: 'You can migrate Docker images previously stored in the Docker registry on {% data variables.product.product_location %} to the {% data variables.product.prodname_container_registry %}.'
product: '{% data reusables.gated-features.packages %}'
permissions: 'Enterprise owners can migrate Docker images to the {% data variables.product.prodname_container_registry %}.'
versions:
  feature: docker-ghcr-enterprise-migration
shortTitle: 迁移到容器注册表
topics:
  - Containers
  - Docker
  - Migration
---

{% data reusables.package_registry.container-registry-ghes-beta %}

## 关于 {% data variables.product.prodname_container_registry %}

{% data reusables.package_registry.container-registry-benefits %} 更多信息请参阅[使用 {% data variables.product.prodname_container_registry %}](/packages/working-with-a-github-packages-registry/working-with-the-container-registry)”。

For more information about configuring {% data variables.product.prodname_registry %} for {% data variables.product.product_location %}, see "[Getting started with {% data variables.product.prodname_registry %} for your enterprise](/admin/packages/getting-started-with-github-packages-for-your-enterprise)."

## 关于从 Docker 注册表迁移

{% data reusables.package_registry.container-registry-replaces-docker-registry %} If the Docker registry on {% data variables.product.product_location %} contains images, you must manually migrate the images to the {% data variables.product.prodname_container_registry %}.

{% ifversion ghes %}

{% note %}

**注**：{% data reusables.package_registry.container-registry-ghes-migration-availability %}

{% endnote %}

{% endif %}

{% data reusables.package_registry.container-registry-migration-namespaces %} For more information about the impact of migration to the {% data variables.product.prodname_container_registry %}, see "[Migrating to the  {% data variables.product.prodname_container_registry %} from the Docker registry](/packages/working-with-a-github-packages-registry/migrating-to-the-container-registry-from-the-docker-registry#about-migration-from-the-docker-registry)."

## Migrating organizations to the {% data variables.product.prodname_container_registry %}

You can start a migration of all your organizations' Docker images to the {% data variables.product.prodname_container_registry %}. The duration of the migration operation depends on the total number of images to migrate, and the overall load on {% ifversion ghes %}your instance{% elsif ghae %}{% data variables.product.product_name %}{% endif %}. After a successful migration, {% data variables.product.product_name %} will display a summary, and all future uploads of Docker images will use the {% data variables.product.prodname_container_registry %}.

If {% ifversion ghes %}a site administrator{% elsif ghae %}an enterprise owner{% endif %} has configured email notifications for {% data variables.product.product_location %}, you will receive an email after the migration is complete. 更多信息请参阅“[配置电子邮件通知](/admin/configuration/configuring-your-enterprise/configuring-email-for-notifications)”。

{% note %}

**{% ifversion ghes %}注释{% elsif ghae %}注释{% endif %}**：

{%- ifversion ghes %}
- During the migration, the CPU and memory usage for your instance will increase. To ensure the performance of the instance for your users, {% data variables.product.company_short %} recommends that you begin a migration during a period of reduced activity.
{%- endif %}
{% ifversion ghes %}- {% endif %}During the migration, do not modify settings for your enterprise{% ifversion ghes %} or run `ghe-config-apply` from an administrative SSH session{% endif %}. {% ifversion ghes %}These actions will trigger a configuration run, which can restart services and {% elsif ghae %}Modifying these settings {% endif %} may interrupt the migration.
{%- ifversion ghes %}
- After the migration, storage pressure on your instance will increase due to the duplication of image files in the Docker registry and the {% data variables.product.prodname_container_registry %}. A future release of {% data variables.product.product_name %} will remove the duplicated files when all migrations are complete.

For more information about monitoring the performance and storage of {% data variables.product.product_location %}, see "[Accessing the monitor dashboard](/admin/enterprise-management/monitoring-your-appliance/accessing-the-monitor-dashboard)."
{% endif %}

{% endnote %}

{% data reusables.enterprise-accounts.access-enterprise %}
{% data reusables.enterprise-accounts.settings-tab %}
1. 在左边栏中，单击 **Packages（包）**。
1. To the right of the number of packages to migrate, click **Start migration**. During the migration, {% data variables.product.product_name %} will display progress on this page.

After the migration completes, the page will display the results. If a migration fails, the page will show the organizations that own the package that caused the failure.

## Re-running a failed organization migration

Prior to migration, if a user has created a package in the {% data variables.product.prodname_container_registry %} that has an identical name to an existing package in the Docker registry, the migration will fail.

1. Delete the affected container in the {% data variables.product.prodname_container_registry %}. 更多信息请参阅“[删除和恢复软件包](/packages/learn-github-packages/deleting-and-restoring-a-package#deleting-a-version-of-an-organization-scoped-package-on-github)”。
{% data reusables.enterprise-accounts.access-enterprise %}
{% data reusables.enterprise-accounts.settings-tab %}
{% data reusables.enterprise-accounts.packages-tab %}
1. To the right of the number of packages to migrate, click **Re-run migration**. During the migration, {% data variables.product.product_name %} will display progress on this page.
1. If the migration fails again, start from step 1 and re-run the migration.
