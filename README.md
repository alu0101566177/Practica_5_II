# Práctica 5 - Google Cardboard

## 1. Configuración Inicial del Proyecto con Google Cardboard

Para comenzar la práctica, se ha seguido la documentación oficial de **Google Cardboard** con el fin de integrar correctamente el SDK en el proyecto Unity y habilitar las funcionalidades básicas de realidad virtual.
La documentación utilizada se encuentra disponible en el siguiente enlace:

[https://developers.google.com/cardboard/develop/unity/quickstart](https://developers.google.com/cardboard/develop/unity/quickstart)

Durante esta fase se configuró:

* El entorno básico de VR.
* La cámara estereoscópica.
* Los ajustes necesarios para la interacción mediante mirada (gaze-based interaction).

Gracias a esta configuración, el proyecto quedó preparado para soportar entrada a través de la dirección de la mirada del usuario y eventos de selección.

## 2. Construcción de la Escena Low Poly

Se desarrolló una escena empleando *assets* de estilo **low poly** que sirven como entorno interactivo. Los recursos utilizados provienen del siguiente enlace:

[https://assetstore.unity.com/packages/3d/environments/landscapes/low-poly-simple-nature-pack-162153](https://assetstore.unity.com/packages/3d/environments/landscapes/low-poly-simple-nature-pack-162153)

La escena incluye varios elementos destacados:

* Diversas **piedras** colocadas en el entorno, que pueden ser seleccionadas mediante la mirada.
* Un **tronco**, que sirve como elemento interactivo principal para desencadenar acciones sobre las piedras seleccionadas.
* Un diseño simplificado y optimizado para rendir adecuadamente en dispositivos móviles compatibles con Cardboard.

## 3. Interacciones Implementadas

### 3.1. Selección de Piedras mediante Mirada

Las piedras dentro de la escena responden a un sistema de interacción basado en gaze.
Cuando el usuario mira fijamente a una piedra, esta queda marcada como “seleccionada”. Se pueden seleccionar varias piedras consecutivamente.

El comportamiento de selección y control de las piedras se implementa mediante el siguiente script:

```csharp
using System.Collections;
using UnityEngine;

public class ColectableItem : MonoBehaviour
{
    public Material defaultMaterial;
    public Material selectedMaterial;
    public Transform player;
    public float selectCooldown = 0.5f;
    public float moveSpeed = 5f;
    public float minDistanceToDisable = 0.1f;

    private Renderer renderer;
    private bool isSelected = false;
    private float nextActionTime = 0f;

    
    public void Start()
    {
        this.renderer = GetComponent<Renderer>();
    }

    public void OnEnable()
    {
        RecolectionObject.OnRecolection += StartMoveToPlayer;
    }

    public void OnDisable()
    {
        RecolectionObject.OnRecolection -= StartMoveToPlayer;
    }

    public void OnPointerClick()
    {
        float currentTime = Time.time;
        if (currentTime < nextActionTime) return;
        ToggleSelect();
        nextActionTime = currentTime + selectCooldown;
    }

    private void ToggleSelect()
    {
        isSelected = !isSelected;
        this.GetComponent<Renderer>().material = isSelected ? selectedMaterial : defaultMaterial;
        
    }

    private void StartMoveToPlayer()
    {
        if (!isSelected) return;
        StartCoroutine(MoveToPlayerRoutine());
    }

    private IEnumerator MoveToPlayerRoutine()
    {
        while (Vector3.Distance(transform.position, player.position) > minDistanceToDisable)
        {
            transform.position = Vector3.MoveTowards(
                transform.position,
                player.position,
                moveSpeed * Time.deltaTime
            );

            yield return null;
        }
        this.gameObject.SetActive(false);
    }
}
```

### 3.2. Evento del Tronco: Movimiento de Piedras hacia el Jugador

El tronco actúa como un activador.
Cuando el usuario fija la mirada sobre él (o realiza la acción asociada de selección), se ejecuta el evento que hace que todas las piedras previamente seleccionadas se desplacen hacia la posición del jugador.

Este comportamiento se implementa mediante el siguiente script:

```csharp
using UnityEngine;
using System;

public class RecolectionObject : MonoBehaviour
{
    public static event Action OnRecolection;

    public void OnPointerClick()
    {
        OnRecolection?.Invoke();
    }
}
```

Esta mecánica permite demostrar de forma práctica:

* La detección de objetos seleccionados mediante mirada.
* La ejecución de una acción global basada en un trigger único.
* Movimientos programados y controlados de objetos dentro de un entorno VR.

## 4. Conclusión

La práctica ha permitido comprender e implementar:

* El proceso de configuración de un proyecto Unity compatible con Google Cardboard.
* La creación de una escena interactiva low poly optimizada para VR en móviles.
* Un sistema de interacción basado en la mirada, tanto para seleccionar objetos como para ejecutar acciones complejas.
* La lógica necesaria para coordinar eventos entre distintos elementos del entorno.

## 5. Inclusión del APK

Como parte de la entrega de la práctica, se incluye el **archivo APK** generado a partir del build final de Unity. Este APK permite instalar y ejecutar la escena VR directamente en un dispositivo Android compatible con Google Cardboard, facilitando la evaluación y prueba sin necesidad de abrir el proyecto en el editor.

[Descargar APK](https://drive.google.com/file/d/17pXU8CcCNuK1QplhfmGnFA_KZJpGPEc2/view?usp=sharing)

La presencia del APK asegura que cualquier usuario o evaluador pueda:

* Probar la experiencia VR de forma inmediata.
* Verificar el correcto funcionamiento de la interacción con las piedras y el tronco.
* Comprobar el rendimiento real en hardware móvil.
