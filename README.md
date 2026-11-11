# Lab de Administración de Active Directory

Implementación y administración de un entorno de Active Directory sobre Windows Server 2025, con dos sucursales (Argentina y Estados Unidos) y dos clientes Windows 11 LTSC unidos al dominio `mydomain.com`.

El laboratorio abarca la organización de usuarios y grupos de seguridad, automatización de tareas mediante PowerShell, configuración de directivas de grupo (GPO) y administración de permisos sobre carpetas compartidas.

![](https://i.imgur.com/UO0o6Qq.png)

## Entorno

- **Server:** Windows Server 2025
- **Clients:** 2 × Windows 11 LTSC
- **Domain:** `mydomain.com`
- **Directory Service:** Active Directory Domain Services (AD DS)
- **Administration Tools:** Active Directory Users and Computers, Group Policy Management
- **Automation:** Windows PowerShell

## Estructura de Active Directory

Se organizó el dominio mediante Unidades Organizativas (OU), separando usuarios y administradores según su sucursal. Además, se creó una OU independiente para centralizar los grupos de seguridad.

<table width="100%" align="center">
  <tr>
    <td align="center">
      <img src="https://i.imgur.com/jTgg82k.png">
      <br>
      <em>Estructura de OUs del dominio.</em>
    </td>
  </tr>
</table>

Se creó la cuenta administrativa `a-anunez` dentro de la OU `_ADMINS` y se incorporó al grupo integrado `Domain Admins`, otorgándole privilegios de administración sobre el dominio.

<table width="100%" align="center">
  <tr>
    <td align="center">
      <img src="https://i.imgur.com/VqDNqCL.png">
      <br>
      <em>Creación de la cuenta administrativa y configuración del nombre de inicio de sesión.</em>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="https://i.imgur.com/yExLWeR.png">
      <br>
      <em>Incorporación de la cuenta a los grupos correspondientes.</em>
    </td>
  </tr>
</table>

## Generación de Usuarios mediante PowerShell

### Preparación de los archivos CSV

Se utilizaron listas públicas de jugadores de las selecciones nacionales de Argentina y Estados Unidos como fuente de datos de prueba.

Los datos se organizaron en archivos CSV con los campos `FirstName` y `LastName`, separados para facilitar su procesamiento mediante PowerShell.

### Desarrollo del script de automatización

Se desarrolló un script de PowerShell para automatizar la creación de usuarios en Active Directory a partir de archivos CSV.

El script contempla:

- Importación de usuarios desde archivos CSV.
- Generación y normalización automática de nombres de usuario, utilizando las iniciales de los nombres y el apellido completo (por ejemplo, Lionel Messi = `lmessi`).
- Detección de conflictos en `sAMAccountName` y UPN.
- Control de identificadores duplicados y generación de alternativas cuando es necesario.
- Selección configurable del archivo CSV y de la OU de destino.
- Generación de archivos de registro (logs) con los resultados de cada ejecución.

<table width="100%" align="center">
  <tr>
    <td align="center" width="50%">
      <img src="https://i.imgur.com/3BOoL1q.png">
      <br>
      <em>Configuración del script de generación de usuarios.</em>
    </td>
    <td align="center" width="50%">
      <img src="https://i.imgur.com/87Toz9b.png">
      <br>
      <em>Archivo CSV utilizado como fuente de datos.</em>
    </td>
  </tr>
</table>

Durante las pruebas se introdujeron deliberadamente registros duplicados y combinaciones de nombres que generaban conflictos, para comprobar los mecanismos de detección y resolución implementados en el script.

<table width="100%" align="center">
  <tr>
    <td align="center">
      <img src="https://i.imgur.com/vsgaVob.png">
      <br>
      <em>Detección de conflictos en identificadores existentes y duplicados dentro de los datos de entrada.</em>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="https://i.imgur.com/FFNVk68.png">
      <br>
      <em>Resultado de la ejecución con los identificadores alternativos generados automáticamente.</em>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="https://i.imgur.com/c7MJw1s.png">
      <br>
      <em>Archivo de registro generado al finalizar el proceso.</em>
    </td>
  </tr>
</table>

## Administración de Grupos Globales

Se crearon grupos globales de seguridad para organizar los usuarios según su sucursal y función dentro del dominio.

Los grupos permiten centralizar la administración de permisos y aplicar restricciones de acceso sin tener que configurar individualmente cada cuenta.

Grupos creados:

- `GG_Branch_Argentina`: usuarios de Argentina.
- `GG_Admins_ARG`: administradores de Argentina.
- `GG_Branch_USA`: usuarios de Estados Unidos.
- `GG_Admins_USA`: administradores de Estados Unidos.

<table width="100%" align="center">
  <tr>
    <td align="center" width="50%">
      <img src="https://i.imgur.com/QEFLjEu.png">
      <br>
      <em>Grupos globales de ambas sucursales.</em>
    </td>
    <td align="center" width="50%">
      <img src="https://i.imgur.com/iHuHdC5.png">
      <br>
      <em>Configuración de la membresía de un grupo de seguridad.</em>
    </td>
  </tr>
</table>

## Administración de Directivas de Grupo (GPO)

Se configuraron diferentes directivas de grupo para personalizar los entornos de trabajo y aplicar restricciones de seguridad según la sucursal.

### Configuración de Wallpapers

Se crearon GPOs para asignar fondos de escritorio diferenciados a los usuarios de Argentina, Estados Unidos y administradores del dominio.

<table width="100%" align="center">
  <tr>
    <td align="center" width="50%">
      <img src="https://i.imgur.com/crGvr3F.png">
      <br>
      <em>GPOs vinculadas a sus respectivas OUs.</em>
    </td>
    <td align="center" width="50%">
      <img src="https://i.imgur.com/HTuUDoo.png">
      <br>
      <em>Configuración de la directiva de wallpaper.</em>
    </td>
  </tr>
</table>

<table width="100%" align="center">
  <tr>
    <td align="center">
      <img src="https://i.imgur.com/27hk1Oj.png">
      <br>
      <em>Verificación de las directivas aplicadas en el cliente.</em>
    </td>
  </tr>
</table>

### Restricciones de Inicio de Sesión

Se configuró la directiva de seguridad `Deny log on locally` para impedir que los usuarios de una sucursal iniciaran sesión de forma interactiva en los equipos asignados a la otra.

Las restricciones se implementaron mediante GPOs vinculadas a las OUs de los equipos y utilizando los grupos globales de seguridad correspondientes.

<table width="100%" align="center">
  <tr>
    <td align="center">
      <img src="https://i.imgur.com/VvAqgZf.png">
      <br>
      <em>Ubicación de los equipos cliente dentro de Active Directory.</em>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="https://i.imgur.com/rRvJ2dO.png">
      <br>
      <em>Vinculación de la GPO a la OU correspondiente a los equipos.</em>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="https://i.imgur.com/fKV94Bl.png">
      <br>
      <em>Configuración de la restricción de inicio de sesión local.</em>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="https://i.imgur.com/AVOv0sr.png">
      <br>
      <em>Verificación de la GPO aplicada en el equipo cliente.</em>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="https://i.imgur.com/uoBmhsy.png">
      <br>
      <em>Prueba de las restricciones de inicio de sesión.</em>
    </td>
  </tr>
</table>

## Carpetas Compartidas y Permisos NTFS

Se crearon carpetas compartidas para separar los recursos de cada sucursal y se configuraron permisos NTFS mediante grupos globales de seguridad.

### Estructura de carpetas

```text
\\DC
│
├── ARG_BRANCH
│
└── USA_BRANCH
```

Las carpetas se compartieron mediante SMB, utilizando los siguientes nombres de recurso compartido:

- `\\DC\ARG_BRANCH`
- `\\DC\USA_BRANCH`

### Configuración de permisos NTFS

Se asignaron permisos diferenciados según la sucursal y la función de cada grupo:

| Carpeta | Grupo de seguridad | Permiso NTFS |
|---|---|---|
| ARG_BRANCH | GG_Branch_ARG | Read |
| ARG_BRANCH | GG_Admins_ARG | Modify |
| USA_BRANCH | GG_Branch_USA | Read |
| USA_BRANCH | GG_Admins_USA | Modify |

De esta manera, los usuarios pueden consultar los recursos de su sucursal, mientras que los administradores cuentan con permisos adicionales para crear, modificar y eliminar archivos y carpetas.

### Configuración de recursos compartidos SMB

Se configuraron los permisos de uso compartido SMB con `Everyone: Full Control`, dejando el control efectivo del acceso a cargo de los permisos NTFS.

<table width="100%" align="center">
  <tr>
    <td align="center">
      <img src="https://i.imgur.com/Ee9qI0N.png">
      <br>
      <em>Estructura y organización de las carpetas compartidas.</em>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="https://i.imgur.com/bE2C003.png">
      <br>
      <em>Configuración de los permisos NTFS.</em>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="https://i.imgur.com/ESJM8oP.png">
      <br>
      <em>Configuración de los permisos de uso compartido SMB.</em>
    </td>
  </tr>
</table>

### Verificación de Acceso

Se realizaron pruebas desde ambos clientes para comprobar que los usuarios pudieran acceder a los recursos correspondientes a su sucursal y que el acceso a los recursos de la otra permaneciera restringido.

También se verificó la diferencia de privilegios entre usuarios estándar y administradores al intentar modificar archivos.

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
      <em>Acceso denegado a los recursos de Estados Unidos.</em>
    </td>
  </tr>
  <tr>
    <td align="center" width="50%">
      <img src="https://i.imgur.com/xI5wYss.png" width="100%">
      <br>
      <em>Modificación de archivos permitida para administradores.</em>
    </td>
    <td align="center" width="50%">
      <img src="https://i.imgur.com/JrtHwqb.png" width="100%">
      <br>
      <em>Modificación de archivos denegada para usuarios estándar.</em>
    </td>
  </tr>
</table>

## Conclusiones y problemas encontrados

Durante el desarrollo del laboratorio se resolvieron algunos inconvenientes relacionados con la configuración de rutas y extensiones de archivos, particularmente al configurar los wallpapers mediante GPO y al seleccionar los archivos CSV para la generación de usuarios.

También se detectaron problemas relacionados con la herencia de permisos NTFS, que fueron solucionados revisando las entradas de permisos y su aplicación sobre las carpetas correspondientes.

El laboratorio permitió poner en práctica conceptos de administración de Active Directory, automatización con PowerShell, aplicación de políticas de grupo y control de acceso a recursos compartidos.

---

**Próximo laboratorio:** [Endpoint Management with Action1](https://github.com/alexisanunez/endpoint-management-action1)
