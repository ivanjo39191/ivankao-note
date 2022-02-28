# Django 呼叫 management 作法

Created: February 25, 2022 1:54 PM
Tags: Django

1.  使用 call_command
    
    ```bash
    from django.core.management import call_command
    
    call_command('clone_tenant', '--clone_from=localhost', '--clone_tenant_fields=True', '--schema_name=156', '--name=156', '--domain-domain=203.70.68.156', '--domain-tenant_id=156', '--domain-is_primary=True')
    ```
    
2. 從要使用的 menagement import Command
    
    ```bash
    from django_tenants.management.commands.clone_tenant import Command
    
    c = Command()
    
    # 範例: 使用 function store_tenant
    tenant_data = {'schema_name': 'example', 'name': 'example', 'description': None}
    
    tenant = c.store_tenant(clone_schema_from=clone_schema_from,
                                           clone_tenant_fields=True,
                                           **tenant_data)
    ```