
# Unity WebGL Browser Interaction Example

This repository demonstrates how to interact between a Unity WebGL build and the browser using JavaScript.

## Communication Reference

Refer to the Unity documentation for detailed guidance on interacting between the browser and Unity: [Unity WebGL Browser Interaction](https://docs.unity3d.com/Manual/web-interacting-browser-unity-to-js.html).

### 1. Create a JSLib File

I've created a JSLib file with the following code. This file allows Unity to interact with browser JavaScript.

```javascript
mergeInto(LibraryManager.library, {
    SendDataToParent: function(data) {
        var jsonString = UTF8ToString(data);
        console.log("JSON string received:", jsonString);

        try {
            var jsonData = JSON.parse(jsonString);
            console.log("Parsed JSON data:", jsonData);
        } catch (e) {
            console.error("Failed to parse JSON:", e);
        }

        window.parent.postMessage(jsonString, "*");
    }, 
    SendExitEventToParent: function() {
        console.log("Sending exit event to parent.");
        var exitEvent = new CustomEvent('unityExitEvent', { detail: { exit: true } });
        window.dispatchEvent(exitEvent);
    },
    SendReplayEventToParent: function() {
        console.log("Sending replay event to parent.");
        var replayEvent = new CustomEvent('unityReplayEvent', { detail: { replay: true } });
        window.dispatchEvent(replayEvent);
    }
});
```

This code defines two JavaScript functions:

1. **`SendDataToParent`**: Receives data from Unity, processes it, and sends it to the parent window.
2. **`SendExitEventToParent`**: Dispatches a custom event (`unityExitEvent`) to notify the browser when the exit action is triggered in Unity.
2. **`SendReplayEventToParent`**: Dispatches a custom event (`unityReplayEvent`) to notify the browser when the replay action is triggered in Unity.

### 2. Call the JavaScript Functions from C#

In my Unity C# script, I can now call the `SendDataToParent`, `SendExitEventToParent` and  `SendReplayEventToParent` functions defined in my JSLib file. The script also provides an example of how to expose a function that JavaScript can call to send data back to Unity.

```csharp
using System.Collections;
using System.Collections.Generic;
using System.Runtime.InteropServices;
using UnityEngine;

[System.Serializable]
public class GameDataDTO
{
    public int points;
}

public class BackendCommunicationManager : MonoBehaviour
{
    public static BackendCommunicationManager Instance { get; private set; }

#if UNITY_WEBGL && !UNITY_EDITOR
    [DllImport("__Internal")]
    private static extern void SendDataToParent(string data);

    [DllImport("__Internal")]
    private static extern void SendExitEventToParent();

    [DllImport("__Internal")]
    private static extern void SendReplayEventToParent();
#endif

    private void Awake()
    {
        if (Instance == null)
        {
            Instance = this;
            DontDestroyOnLoad(gameObject);
        }
        else
        {
            Destroy(gameObject);
        }
    }

    public void SendPlayerDataToParent(int playerPoints)
    {
        GameDataDTO data = new GameDataDTO
        {
            points = playerPoints
        };

        string jsonData = JsonUtility.ToJson(data);

#if UNITY_WEBGL && !UNITY_EDITOR
    SendDataToParent(jsonData);
#else
        Debug.Log("JSON data being sent: " + jsonData);
        Debug.Log("SendDataToParent is only available in WebGL builds.");
#endif
    }


    public void SendExitEvent()
    {
#if UNITY_WEBGL && !UNITY_EDITOR
        SendExitEventToParent();
#else
        Debug.Log("SendExitEventToParent is only available in WebGL builds.");
#endif
    }

    public void SendReplayEvent()
    {
#if UNITY_WEBGL && !UNITY_EDITOR
        SendReplayEventToParent();
#else
        Debug.Log("SendReplayEventToParent is only available in WebGL builds.");
#endif
    }

}

```

### 3. Handling Exit Events in the Browser

To handle exit events in the browser, you can add a listener for the `unityExitEvent` custom event. This event can be used to trigger actions such as redirecting the user or reloading the page.

```javascript
// Example of handling the unityExitEvent in JavaScript
window.addEventListener('unityExitEvent', function(event) {
    console.log("Exit event received from Unity:", event.detail);
    if (event.detail.exit) {
        location.reload(); // Example action: reload the page
    }
});
```

This listener:

- **`unityExitEvent`**: Listens for the custom `unityExitEvent` dispatched by Unity when the user triggers an exit action.
- **`event.detail.exit`**: Checks if the event contains an `exit: true` value and performs an action based on that (e.g., reloading the page).


### Summary

- **Function Exposure**: The `ReceivePlayerData` function in the Unity script is exposed to allow JavaScript to send data back to Unity. 
- **Sending Data**: Use the `SendMessage` function in JavaScript to interact with Unity by calling the exposed method and passing the required data.
- **Exit Event Handling**: The `SendExitEventToParent` function and the corresponding event listener in the browser provide a way to handle exit actions triggered from Unity, allowing for custom behavior like page reloads or redirects.
