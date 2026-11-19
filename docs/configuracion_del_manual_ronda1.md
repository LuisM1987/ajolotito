# Ronda 1. Preparar el proyecto
```
$ flutter create ajolotito
$ cd ajolotito
```
Sustituye pubspec.yaml por el siguiente (o agrega las dependencias con flutter pub add). Observa la restricción de versión del plugin: ">=20.1.0 <24.0.0" porque desde la 20 las firmas son con nombre.

```
📄 pubspec.yaml
name: ajolotito
description: Ajolotito hambriento - notificaciones locales bien hechas en Flutter.
publish_to: 'none'
version: 1.0.0+1

environment:
  sdk: ^3.8.0

dependencies:
  flutter:
    sdk: flutter
  # Desde la v20 los parametros de show/zonedSchedule/cancel/initialize son con nombre.
  flutter_local_notifications: ">=20.1.0 <24.0.0"
  timezone: ^0.10.0
  shared_preferences: ^2.3.0
  app_settings: ^5.1.1

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^5.0.0

flutter:
  uses-material-design: true
```
```
$ flutter pub get
```

## Configuración de Android (una sola vez)
Archivo android/app/src/main/AndroidManifest.xml. Dentro de <manifest> (fuera de <application>):

```
<uses-permission android:name="android.permission.POST_NOTIFICATIONS"/>
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
```
Dentro de <application>, los receptores que permiten que las notificaciones programadas funcionen y sobrevivan a un reinicio del teléfono:

```
<receiver android:exported="false"
    android:name="com.dexterous.flutterlocalnotifications.ScheduledNotificationReceiver" />
<receiver android:exported="false"
    android:name="com.dexterous.flutterlocalnotifications.ScheduledNotificationBootReceiver">
  <intent-filter>
    <action android:name="android.intent.action.BOOT_COMPLETED"/>
    <action android:name="android.intent.action.MY_PACKAGE_REPLACED"/>
    <action android:name="android.intent.action.QUICKBOOT_POWERON" />
    <action android:name="com.htc.intent.action.QUICKBOOT_POWERON"/>
  </intent-filter>
</receiver>
```
Archivo android/app/build.gradle.kts (o build.gradle en proyectos viejos): el plugin usa funciones modernas de Java que requieren "desugaring":

```
android {
    compileOptions {
        isCoreLibraryDesugaringEnabled = true
    }
}
dependencies {
    coreLibraryDesugaring("com.android.tools:desugar_jdk_libs:2.1.4")
}
```
⚠️ Cuidado

Las versiones exactas (desugar_jdk_libs, nivel de Java, compileSdk) las dicta el README del paquete para la versión que instalaste; si el compilador de Android se queja, ahí está la respuesta. No las copies a ciegas de este manual.


## Configuración de iOS
En ios/Runner/AppDelegate.swift, dentro de application(_:didFinishLaunchingWithOptions:), antes de return, agrega la línea que permite mostrar notificaciones y recibir toques:

```
UNUserNotificationCenter.current().delegate = self as? UNUserNotificationCenterDelegate
```
Las versiones recientes del paquete también documentan cambios por el nuevo ciclo de vida UIScene de iOS; sigue la sección de iOS del README. En iOS los permisos se piden en tiempo de ejecución (lo haremos en la Ronda 8) y las notificaciones programadas pendientes tienen un máximo de 64 por app: otra razón para programar solo lo necesario.


## Verifica el punto de partida
```
$ flutter test
```
Todavía no hay pruebas, así que no verás nada. Vamos a escribirlas primero.

