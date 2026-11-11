# Lab de Administración de Active Directory

Administración de un entorno de Active Directory sobre Windows Server 2025, con dos sucursales (Argentina y Estados Unidos) y dos clientes Windows 11 LTSC unidos al dominio `mydomain.com`.

El laboratorio incluye organización de usuarios y grupos de seguridad, automatización mediante PowerShell, configuración de directivas de grupo (GPO) y administración de permisos sobre carpetas compartidas.

![](https://i.imgur.com/UO0o6Qq.png)

## Entorno

- **Server:** Windows Server 2025
- **Clients:** 2 × Windows 11 LTSC
- **Domain:** `mydomain.com`
- **Directory Service:** Active Directory Domain Services (AD DS)
- **Administration Tools:** Active Directory Users and Computers, Group Policy Management
- **Automation:** Windows PowerShell

## Estructura de Active Directory

Se organizó el dominio mediante Unidades Organizativas (OU), separando usuarios y administradores según su sucursal. También se creó una OU independiente para centralizar los grupos de seguridad.

<table width="100%" align="center">
  <tr>
    <td align="center">
      <img src="https://i.imgur.com/jTgg82k.png" >
      <br>
      <em>Estructura de OUs del dominio.</em>
    </td>
  </tr>
</table>

Se creó la cuenta administrativa a-anunez dentro de la OU _ADMINS y se incorporó al grupo integrado Domain Admins, otorgándole permisos de administración sobre el dominio.

<table width="100%" align="center">
  <tr>
    <td align="center" width="50%">
      <img src="https://i.imgur.com/VqDNqCL.png" >
      <br>
      <em>Creación de la cuenta administrativa y vista del logon name.</em>
    </td>
  </tr>
</table>

<table width="100%" align="center">
  <tr>
    <td align="center" width="50%">
      <img src="https://i.imgur.com/yExLWeR.png" >
      <br>
      <em>Incorporación del usuario a los grupos necesarios.</em>
    </td>
  </tr>
</table>

## Generación de Usuarios mediante PowerShell

### Creación del archivo CSV con los nombres de los usuarios a generar

Para los archivos CSV con los nombres decidí descargar las listas jugadores de la selección de fútbol respectiva al país de cada branch (en este caso Argentina y Estados Unidos). Y utilicé la IA para crear el formato del archivo necesario para el script (nombres y apellidos separados por una coma, en otros idiomas puede ser confuso) de una manera eficiente y con el menor riesgo de error posible.

### Desarrollo de un script de PowerShell para automatizar la creación de usuarios en Active Directory a partir de archivos CSV.

El script contempla:

- Importación de usuarios desde archivos CSV.
- Generación y normalización automática de nombres de usuario (utiliza la primera letra del nombre + apellido completo, por ej. Lionel Messi = lmessi).
- Detección de conflictos en sAMAccountName y UPN.
- Control de nombres duplicados y modificación de identificadores cuando es necesario.
- Selección configurable del archivo CSV y OU de destino.
- Generación de logs con los resultados de ejecución.

<table width="100%" align="center">
  <tr>
    <td align="center">
      <img src="https://i.imgur.com/3BOoL1q.png" >
      <br>
      <em>Configuración del script de generación de usuarios.</em>
    </td>
    <td align="center">
      <img src="https://i.imgur.com/87Toz9b.png" >
      <br>
      <em>Archivo CSV utilizado como fuente de datos.</em>
    </td>
  </tr>
</table>

<table width="100%" align="center">
  <tr>
    <td align="center">
      <img src="https://i.imgur.com/vsgaVob.png" >
      <br>
      <em>Detección y corrección de conflictos del script, véase que "efernandez" se detecta duplicado debido a combinaciones similares, tambien ya hay un "lmessi" creado en AD, por lo que agregué uno más a la lista de generación e incluso un "Leonardo Messi" para verificar el funcionamiento de detección de duplicados.</em>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="https://i.imgur.com/FFNVk68.png" >
      <br>
      <em>Resultado de la ejecución del script con los nombres corregidos automaticamente.</em>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="https://i.imgur.com/c7MJw1s.png" >
      <br>
      <em>Log file generado luego del proceso de creación de usuarios.</em>
    </td>
  </tr>
</table>

## Administración de grupos globales

- Creación de grupos globales para organizar usuarios según sucursal y función.
- Asignación de usuarios a sus respectivos grupos de seguridad.
- Utilización de los grupos para administrar permisos sobre carpetas compartidas y restricciones de inicio de sesión.
<table width="100%" align="center">
  <tr>
    <td align="center" width="50%">
      <img src="https://i.imgur.com/QEFLjEu.png" >
      <br>
      <em>Grupos globales para ambas sucursales.</em>
    </td>
    <td align="center" width="50%">
      <img src="https://i.imgur.com/iHuHdC5.png" >
      <br>
      <em>Membresía de un grupo de seguridad.</em>
    </td>
  </tr>
</table>


## Administración de Directivas de Grupo (GPO)

Se configuraron diferentes políticas de grupo para personalizar los entornos de trabajo y aplicar restricciones de seguridad según la sucursal.

### Configuración de Wallpapers

Creación de GPOs para asignar fondos de escritorio diferenciados a usuarios de Argentina, Estados Unidos y administradores del dominio.

<table width="100%" align="center">
  <tr>
    <td align="center" width="50%">
      <img src="https://i.imgur.com/crGvr3F.png" >
      <br>
      <em>GPOs linkeadas en sus respectivas OUs.</em>
    </td>
    <td align="center" width="50%">
      <img src="https://i.imgur.com/HTuUDoo.png" >
      <br>
      <em>Configuración de la política de wallpaper.</em>
    </td>
  </tr>
</table>



<table width="100%" align="center">
  <tr>
    <td align="center">
      <img src="https://i.imgur.com/27hk1Oj.png" >
      <br>
      <em>Verificación de las políticas aplicadas.</em>
    </td>
  </tr>
</table>

### Restricciones de Inicio de Sesión

Configuración de la directiva `Deny log on locally` para impedir que usuarios de una sucursal inicien sesión localmente en los equipos asignados a la otra.

Las restricciones se implementaron mediante GPO y grupos globales de seguridad.

<table width="100%" align="center">
  <tr>
    <td align="center">
      <img src="https://i.imgur.com/VvAqgZf.png" >
      <br>
      <em>Ubicación de los clientes en AD, en este caso los tengo en "Action1 Deployment".</em>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="https://i.imgur.com/rRvJ2dO.png" >
      <br>
      <em>Vinculación de la GPO a la OU de los equipos de la sucursal.</em>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="https://i.imgur.com/fKV94Bl.png" >
      <br>
      <em>Configuración de la restricción de inicio de sesión local.</em>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="https://i.imgur.com/AVOv0sr.png" >
      <br>
      <em>Verificación de la GPO aplicada.</em>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="https://i.imgur.com/uoBmhsy.png" >
      <br>
      <em>Verificación de las restricciones de inicio de sesión.</em>
    </td>
  </tr>
</table>

## Carpetas Compartidas y Permisos NTFS

Creación de carpetas compartidas para separar los recursos de cada sucursal y configuración de permisos NTFS mediante grupos globales de seguridad.

Estructura de carpetas:

```text
\\DC\
│
├── ARG_BRANCH
│
└── USA_BRANCH
```

Los permisos se configuraron de la siguiente manera:

| Carpeta | Grupo de seguridad | Permiso NTFS |
|---|---|---|
| ARG_BRANCH | Argentina Users | Read |
| ARG_BRANCH | Argentina Admins | Modify |
| USA_BRANCH | USA Users | Read |
| USA_BRANCH | USA Admins | Modify |

<table width="100%" align="center">
  <tr>
    <td align="center">
      <img src="https://i.imgur.com/Ee9qI0N.png" >
      <br>
      <em>Organización de los recursos compartidos por sucursal.</em>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="https://i.imgur.com/bE2C003.png" >
      <br>
      <em>Configuración de permisos NTFS.</em>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="https://i.imgur.com/ESJM8oP.png" >
      <br>
      <em>Compartir carpetas y configurar permisos SMB (Full control a "everyone" porque el acceso ya será controlado mediante los permisos NTFS).</em>
    </td>
  </tr>
</table>

Verificación de Acceso
Se realizaron pruebas desde ambos clientes para comprobar que los usuarios pudieran acceder a los recursos de su sucursal y que el acceso a los recursos de la otra permaneciera restringido.

<table width="100%" align="center">
  <tr>
    <td align="center" width="50%">
      <img src="https://i.imgur.com/P1qX3fV.png" width="100%">
      <br>
      <em>Acceso autorizado a los recursos de Argentina.</em>
    </td>
    <td align="center" width="50%">
      <img src="https://i.imgur.com/lfAowcw.png" width="100%">
      <br>
      <em>Acceso restringido a los recursos de Estados Unidos.</em>
    </td>
  </tr>
  <tr>
    <td align="center" width="50%">
      <img src="https://i.imgur.com/xI5wYss.png" width="100%">
      <br>
      <em>Acceso autorizado para modificar elementos a los admins.</em>
    </td>
    <td align="center" width="50%">
      <img src="https://i.imgur.com/JrtHwqb.png width="100%">
      <br>
      <em>Acceso restringido para modificar elementos a las cuentas de user (se intentó guardar el archivo editado).</em>
    </td>
  </tr>
</table>

## Conclusiones y problemas encontrados en el lab

No hubo complicaciones mayores, algunos los errores encontrados fueron por no agregar la extensión de los archivos cuando indicabas su nombre (por ejemplo el el GPO de los fondos de pantalla y al seleccionar el archivo de nombres del script de generación). Y otros por herencia de permisos en las carpetas que fueron solucionados rapidamente.

Próximo lab: ### [Endpoint Management with Action1](https://github.com/alexisanunez/endpoint-management-action1). Saludos!
