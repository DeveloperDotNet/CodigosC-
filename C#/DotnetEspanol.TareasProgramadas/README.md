## Tareas programadas

Una tarea programada es aquella que deseamos que se realice en cierto momento
y que no nos preocupemos por que se ejecute.

En el caso de las aplicaciones web, este tipo de actividades se vuelve
m�s complicada por el simple hecho de que la vida de una p�gina se mide
por solicitudes, osea que inicia y termina con una solicitud.

Una soluci�n simple es guardar en la base de datos la tarea y su fecha de
"�ltima ejecuci�n" y en el c�digo consultar esa fecha y si es anterior a
la actual, entonces ejecutar la tarea. Pero a�n as� existe un problema con
esa soluci�n, y es que es posible que se ejecute por cada solicitud, y si
en ese momento entran varios, pues se ejecutar� varias veces la misma tarea.

## �C�mo programar una tarea?

**FluentSheduler** es un "programador de tareas autom�ticas" y es el usado en
este ejemplo. Para programar una tarea se debe hacer dos cosas.

- Crear la tarea
- Registrar cu�ndo se ejecutar�.

Para **crear la tarea** observe el archivo [Core\Jobs\SendEmail.cs](DotnetEspanol.TareasProgramadas/Core/Jobs/SendEmail.cs)
y modifique el siguiente bloque de c�digo:

```
/// <summary>
/// Cambia el estado de las solicitudes que no han sido aceptadas al �rea de oportunidad.
/// </summary>
public class SendEmail : IJob, IRegisteredObject
{
    /// <summary>
    /// Realiza la tarea.
    /// </summary>
    private void DoWork()
    {
        ...
    }
}
```

> La interfaz `IJob` requiere implementar el m�todo `void Execute()`, este
es el que llama a la funci�n `DoWork()`.

> La clase `SendEmail` implementa `IRegisteredObject`, lo cu�l permite
controlar cuando IIS est� cerrando la aplicaci�n.

Una vez hecho eso, se debe **registrar cu�ndo se ejecutar�** y para ello
usamos una clase llamada [JobRegistry.cs](DotnetEspanol.TareasProgramadas/Core/Jobs/JobRegistry.cs)

Es ah� donde nosotros modificaremos la frecuencia con la que se ejec�tar�
la tarea programada.

```
// Se ejecuta "ahora" y cada 1 d�a a las 11:00 horas.
Schedule<SendEmail>().ToRunNow().AndEvery(1).Days().At(11, 00);
```

## Global.asax

El archivo [Global.asax](DotnetEspanol.TareasProgramadas/Global.asax.cs)
es el que contiene la clase de entrada para la aplicaci�n. En el existe el m�todo 
`void Application_Start()`, que es el primero que se ejecutar�.

Es ah� donde daremos inicio con todas las tareas programadas con el siguiente
c�digo:

```
// Configurar las tareas programadas del sistema.
JobManager.Initialize(new Core.Jobs.JobRegistry());
```

## Consideraciones

Puede consultar las diferentes formas de configurar las tareas programadas
en la p�gina de [FluentSheduler](https://github.com/fluentscheduler/FluentScheduler).

Una aplicaci�n web se recicla cada 29 horas, eso quiere decir que IIS cierra
los hilos creados para la aplicaci�n y los vuelve a crear. Es por eso que 
es mejor utilizar `ToRunNow()` para ejecutar las tareas. Adicional a esto,
si nadie entra a la p�gina por ~20 minutos, entonces IIS la pone a "dormir".
Por tal motivo es probable que no se ejecute la tarea.

Puede hacer una tarea cada 19 minutos por ejemplo, y que esta llame a cualquier
p�gina de la aplicaci�n usando un `WebClient`, eso mantendr� "despierta" a la aplicaci�n.

Hay aplicaciones que corren en "granjas de servidores" (no es lo mismo que un "jard�n")
esto significa que la misma tarea se puede ejecutar varias veces, en ese caso
podemos cambiarnos a [HangFire](hangfire.com) para controlar mejor eso.

---
DotNetEspanol