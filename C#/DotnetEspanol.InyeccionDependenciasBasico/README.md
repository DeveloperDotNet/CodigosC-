# Inyecci�n de dependencias

La inyecci�n de dependencias no es m�s que simplemente una forma
de programar sin necesidad de qui�n realice la tarea ni c�mo, 
s�lo programar.

Imagine que su c�digo tiene una clase llamada `ConexionDb`. Esa la usa
en cada parte de la aplicaci�n, sin embargo esta usa SqlServer, y esta
fuertemente ligada a SqlServer. Ya ha programado decenas de clases que se
conectan e inclusive no est� seguro de d�nde se usa.

En alg�n momento el cliente decide cambiar a MySql, �cu�nto tiempo le tomar�a
cambiar el c�digo?, digamos que en 1 hora. Si hubiera usado la inyecci�n
de dependencias, le tomar�a unos segundos (o minutos si su computadora fuera tan lenta como la m�a).

El patr�n de "inyecci�n de dependencias" s�lo establece qu� deber�a de hacer
una clase como `ConexionDb` mas no como lo hace (implementa). Es ah� donde
reside su importancia. Ya que las dem�s clases, no requieren acoplarse
o vincularse con ninguna en especifico, solamente con la "interfaz".

Hay muchos art�culos sobre inyecci�n de dependencias, en este ejemplo no se
trata de ense�ar a detalle c�mo funciona, pero si observan las clases
contenidas en el proyecto se dar�n una idea de c�mo funciona internamente.

## Paso 1: Definir la actividad.

Una interfaz define el nombre de las actividades. Por lo cual hay que crear
primero una interfaz.

```csharp
public interface IEmailService
{
    string GetServiceName();
    void SendEmail(string to, string subject, string message);
}
```

Ah� estamos definiendo que debe existir una clase que tenga la funci�n
de enviar con esos parametros.

## Paso 2: Implementar la interfaz.

Implementar una interfaz, no es m�s que crear una clase que "implemente"
dicha interfaz, osea tenga los mismos m�todos (firmas) que la interfaz.

```csharp
public class NonEmailService : IEmailService
{
    public void SendEmail(string to, string subject, string message)
    {
        // Esta funci�n no har� nada por el momento.
    }
}
```

## Paso 3: Consumir el servicio.

En este punto, la clase `HomeController` no conoce qui�n va a implementar
la funcionalidad. Simplemente creamos una instancia por medio de `GetService`.

Observa c�mo funciona `ServiceLocator.GetService<T>();`, quiz� sea algo complicado,
pero en si no lo es, simplemente crea y devuelve una instancia.

```csharp
public class HomeController : Controller
{
    public ActionResult Index()
    {
        // Obtener la clase que se encargar� de realizar el envio de correos.
        var _email = ServiceLocator.GetService<IEmailService>();

        // Enviar el correo.
        _email.SendEmail("foo@bar.com", "Saldo vencido.", "Le informamos que su saldo se ha vencido.");

        // Esta p�gina no tiene ninguna l�gica.
        return this.View();
    }
}
```

## Paso 4: Registrar la instancia.

Antes de poder usarlas, las clases deben ser registradas en algo
que se le llama "Contenedor", en este caso, nuestro contenedor se llama
`ServiceLocator` (puede ser cualquier nombre).

Vamos a usar siempre `Global.asax.cs` para registrar los servicios, porque
es lo que se ejecuta siempre de primero.

```csharp
protected void Application_Start()
{
    // Registrar qu� instancia enviar� el correo.
    ServiceLocator.AddScope<IEmailService>(typeof(NonEmailService));
}
```

## Consideraciones.

Es importante conocer este patr�n, con .NET Core, es algo de suma
importancia.

Esta implementaci�n de DI, es una forma b�sica de aprender sobre c�mo
funciona, sin embargo hay bibliotecas m�s elaboradas y profesionales
como Ninject e inclusive el nuevo DependencyInjection de .NET.

Cambie en el `Global.asax` de `NonEmailService` a `EmailService` para
ver como funciona. 

> Recuerda que puedes instalar [Papercut](https://github.com/changemakerstudios/papercut) 
para recibir y leer los correos.

---------
DotnetEspanol