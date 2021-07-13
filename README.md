# ByNameModding
Modding (hacking) il2cpp games by classes, methods, fields names on Android.

# Dependencies
SubstrateHook (MSHook) for Il2CppRegistartion and MetadataRegistartion finder (only armeabi-v7a)

# Unity support
| Il2Cpp Version | Unity Version     |
| -------------- | ----------------- |
| 24.0           | 2017.x - 2018.2.x |
| 24.1           | 2018.3.x          |
| 24.2           | 2019.x            |
| 27.0           | 2020.x - 2021.x   |

## You need to select Unity version in ByNameModding.h

# Usage
## get_method example
```c++
/* get_method: edit fov example
* code from here
* Il2CppResolver
* https://github.com/MJx0/IL2CppResolver/blob/master/Android/test/src/demo.cpp
* MJx0's IL2CppResolver doesn't work
* get_method working ONLY with extren methods
*/
void *set_fov(float value) {
    int (*Screen$$get_height)();
    int (*Screen$$get_width)();
    InitResolveFunc(Screen$$get_height, "UnityEngine.Screen::get_height()"); // #define InitResolveFunc(x, y)
    InitResolveFunc(Screen$$get_width, "UnityEngine.Screen::get_width()");
    if (Screen$$get_height && Screen$$get_width) {
        LOGI("%dx%d", Screen$$get_height(), Screen$$get_width());
    }

    uintptr_t (*Camera$$get_main)(); // you can use void *
    float (*Camera$$get_fieldofview)(uintptr_t);
    void (*Camera$$set_fieldofview)(uintptr_t, float);

    InitResolveFunc(Camera$$get_main, "UnityEngine.Camera::get_main()");
    InitResolveFunc(Camera$$set_fieldofview, "UnityEngine.Camera::set_fieldOfView(System.Single)");
    InitResolveFunc(Camera$$get_fieldofview, "UnityEngine.Camera::get_fieldOfView()");

    if (Camera$$get_main && Camera$$get_fieldofview && Camera$$set_fieldofview) {
        uintptr_t mainCamera = Camera$$get_main();
        if (mainCamera != 0) {
            float oldFOV = Camera$$get_fieldofview(mainCamera);
            Camera$$set_fieldofview(mainCamera, value);
            float newFOV = Camera$$get_fieldofview(mainCamera);
            LOGI("Camera Ptr: %p  |  oldFOV: %.2f  |  newFOV: %.2f", (void *) mainCamera, oldFOV,
                 newFOV);
        } else {
            LOGE("mainCamera is currently not available!");
        }
    }
}
```
## LoadClass and Field exampels
```c++

//! Create new class exampele
LoadClass GameObject;
LoadClass ExampleClass;
void *NewExampleClass(){
    void *new_GameObject = GameObject.CreateNewClass({}, 0); // No args
    bool ExampleArg1 = false;
    float ExampleArg2 = 0.f;
    void* args[] = {&ExampleArg1, &ExampleArg2, new_GameObject};
    void *new_ExampleClass = ExampleClass.CreateNewClass(args, 3); // 3 args
    return new_ExampleClass;
}

//! Find exampele
void *(*get_Transform)(void *instance);
void (*set_position)(void *Transform, Vector3);
void *myPlayer;
void (*old_Update)(void *instance);
void Update(void *instance){
    old_Update(instance);
    /** We have: public static FPSControler LocalPlayer; **/
	myPlayer = FieldBNStatic(void *, "", "FPSControler", false, "LocalPlayer"); // false - old method
    void *myPlayer_Transform = get_Transform(myPlayer);
    set_position(myPlayer_Transform, Vector3(0, 0, 0));
}

//! Find using metadata exampele
void *instance_from_IEnumerator;
bool (*old_MainLoop$$MoveNext)(void *instance);
bool MainLoop$$MoveNext(void *instance) {
    LOGI("MainLoop$$MoveNext");
	instance_from_IEnumerator = FieldBNC(void *, instance, "", "<MainLoop>d__1", true, "<>4__this"); // true - new method
    return old_MainLoop$$MoveNext(instance);
}

void *hack_thread(void *) {
    do {
        sleep(1);
    } while (!isLibraryLoaded(libName));
	
	
    //! Create new class exampele
    GameObject = LoadClass("UnityEngine", "GameObject");
    ExampleClass = LoadClass("", "ExampleClass");
	
	
    //! Find exampele
    auto Transform = LoadClass("UnityEngine", "Transform");
    auto Component = LoadClass("UnityEngine", "Component");
    auto FPSControler = LoadClass("", "FPSControler");
	
    InitFunc(get_Transform, Component.GetMethodOffsetByName("get_transform", 0)); // 0 - parameters count in original c# method
    InitFunc(set_position,  Transform.GetMethodOffsetByName("set_position_Injected", 1); // set_position does not work well, so we use set_position_Injected
	
    MSHookFunction((void *)FPSControler.GetMethodOffsetByName("Update", 0), (void *) Update, (void **) &old_Update);
	
	
    //! Find using metadata exampele
	
    /** Class generated by IEnumerator MainLoop(); **/
    auto MainLoop_d1 = LoadClass("", "<MainLoop>d__1", true); // true - new method to find class
    // Only using new method you can get class with CompilerGeneratedAttribute
	
    MSHookFunction((void *)MainLoop_d1.GetMethodOffsetByName("MoveNext", 0), (void *) MainLoop$$MoveNext, (void **) &old_MainLoop$$MoveNext);
	
	
    //! Find metohod by name and parameters names
    /** 
    In UnityEngine.Physics we have 16 Raycast methods
    Some have the same number of parameters.
    For example:
    We need:
    Raycast(Ray ray, out RaycastHit hitInfo)
    but LoadClass finds by number of parameters:
    Raycast(Vector3 origin, Vector3 direction)
    Now this is not a problem.
    **/
    const char *Raycast_Params[] = {"ray", "hitInfo"};
    auto RayCastOffset = LoadClass("UnityEngine", "Physics", false).GetMethodOffsetByName("Raycast", Raycast_Params);
    
    
    //! Find Class by name and method name
    //! GetLC_ByClassAndMethodName
    /**
    Example from among us:
    We have HatManager class and <>c class in it.
    To get in il2cpp 
    In Il2Cpp the class is named not like this:
    HatManager.<>c
    Like this:
    <>c
    And to Get it you need use:
    GetLC_ByClassAndMethodName
    **/
    // Then you can get any method
    auto HatManager_c = LoadClass::GetLC_ByClassAndMethodName("", "<>c", "<GetUnlockedHats>b__10_0");
}
```
