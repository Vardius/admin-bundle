Vardius - Admin Bundle
======================================

Configuration
----------------
1. Configure the VardiusAdminBundle
2. Add items to admin menu
3. Create crud actions under admin route
4. Use i18n_prefix for routes

### 1. Configure the VardiusAdminBundle

If you want to enable username
config.yml

``` yaml
    #app/config/config.yml
    
    twig:
        form:
            resources: ['bootstrap_3_horizontal_layout.html.twig']
    
    vardius_user:
        username: true #default false
        email_from: some@email.com #default hostname
```
        
routing.yml

``` yaml
    #app/config/routing.yml
    
    vardius_admin:
        resource: "@VardiusAdminBundle/Resources/config/routing.yml"
        prefix:   /
```
        
security.yml

``` yaml
    #app/config/security.yml
    
    encoders:
        Vardius\Bundle\UserBundle\Entity\User:
            algorithm: bcrypt
            cost: 12

    role_hierarchy:
        ROLE_ADMIN:       ROLE_USER
        ROLE_SUPER_ADMIN: [ ROLE_ADMIN, ROLE_ALLOWED_TO_SWITCH ]

    providers:
        vardius:
            id: vardius_user.user_provider

    firewalls:
        admin_area:
            pattern:    ^/admin
            anonymous: ~
            form_login:
                login_path: login_route
                check_path: login_check
                csrf_provider: form.csrf_provider
            logout:
                path:   logout_route
                target: login_route
                invalidate_session: true
            remember_me:
                key:      "%secret%"
                lifetime: 31536000 # 365 days in seconds
                path:     /admin
                domain:   ~ # Defaults to the current domain from $_SERVER

    access_control:
        - { path: ^/admin/login, roles: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/admin/register, roles: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/admin/password-reset, roles: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/admin/terms, roles: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/admin/reset, roles: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/admin, roles: ROLE_ADMIN }
        - { path: ^/, roles: IS_AUTHENTICATED_ANONYMOUSLY }
```

### 2. Add items to admin menu
Follow the steps from documentation [Vardius Menu Bundle](https://github.com/Vardius/menu-bundle)
add menu items to `admin_menu`

``` php
    //AcmeBundle\EventListener\MenuSubscriber
    
    public static function getSubscribedEvents()
    {
        return array(
            MenuEvents::MENU => 'onMenu',
        );
    }
    
    public function onMenu(MenuEvent $event)
    {
        $menu = $event->getMenu();
        if ($menu->getName() === 'admin_menu') {
    
            $menu->addChild(new MenuItem('Page Name', 'page_path'));
        }
    }
```

### 3. Create crud actions under admin route
Follow the steps from documentation [Vardius User Bundle](https://github.com/Vardius/user-bundle)

use `vardius_admin.controller.factory` as a factory service

``` xml
    <service id="acme.crud_controller.product" class="%vardius_crud.controller.class%" factory-service="vardius_admin.controller.factory" factory-method="get">
        <argument>AcmeBundle:Product</argument>
        <argument>/products</argument>
        <argument type="service" id="acme.product.list_view"/>
        <argument type="service" id="acme.form.type.product"/>
    
        <tag name="vardius_crud.controller" />
    </service>
```

and admin actions to use view from admin bundle for example:

``` xml
    <argument type="collection">
        <argument type="service" key="show" id="vardius_admin.action_show"/>
        <argument type="service" key="edit" id="vardius_admin.action_edit"/>
    </argument>
```

### 4. Use i18n_prefix for routes
Change routes as follows

``` yml
vardius_admin:
    resource: "@VardiusAdminBundle/Controller/"
    type:     annotation
    prefix:   /
    options: { i18n_prefix: admin }

vardius_user:
    resource: "@VardiusUserBundle/Resources/config/routing.yml"
    prefix:   /
    options: { i18n_prefix: admin }
```
