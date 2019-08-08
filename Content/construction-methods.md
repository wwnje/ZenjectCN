# 构造方法  Construction Methods

1. **FromNew**  - 通过C＃new运算符创建。 如果未指定构造方法，则这是默认值。

    ```csharp
    // These are both the same
    Container.Bind<Foo>();
    Container.Bind<Foo>().FromNew();
    ```

2. **FromInstance**  - 将一个给定的实例添加到容器中。 请注意，在这种情况下不会注入给定的实例。 如果您还希望在启动时注入实例，请参阅[QueueForInject](dicontainer-methods-queueforinject)

    ```csharp
    Container.Bind<Foo>().FromInstance(new Foo());

    // 您也可以使用这个简短的参与者从参数类型中获取ContractType
    Container.BindInstance(new Foo());

    // 这也是您通常用于原始类型的内容
    Container.BindInstance(5.13f);
    Container.BindInstance("foo");

    //或者，如果您有许多实例，则可以使用BindInstances
    Container.BindInstances(5.13f, "foo", new Foo());
    ```

3. **FromMethod**  - 通过自定义方法创建

    ```csharp
    Container.Bind<Foo>().FromMethod(SomeMethod);

    Foo SomeMethod(InjectContext context)
    {
        ...
        return new Foo();
    }
    ```

1. **FromMethodMultiple**  - 与FromMethod相同，但允许一次返回多个实例（或零）。

    ```csharp
    Container.Bind<Foo>().FromMethodMultiple(GetFoos);

    IEnumerable<Foo> GetFoos(InjectContext context)
    {
        ...
        return new Foo[]
        {
            new Foo(),
            new Foo(),
            new Foo(),
        }
    }
    ```

1. **FromFactory**  - 使用自定义工厂类创建实例。 这种构造方法类似于`FromMethod`，除非在逻辑更复杂或需要依赖性的情况下更清晰（因为工厂本身可以注入依赖项）

    ```csharp
    class FooFactory : IFactory<Foo>
    {
        public Foo Create()
        {
            // ...
            return new Foo();
        }
    }

    Container.Bind<Foo>().FromFactory<FooFactory>()
    ```

1. **FromIFactory**  - 使用自定义工厂类创建实例。 这是FromFactory的一个更通用和更强大的替代方法，因为你可以使用任何你想要的自定义工厂的构造方法（不像FromFactory假定`FromNew().AsCached()`）

    例如，您可以使用像这样的可编写脚本的对象的工厂：

    ```csharp
    class FooFactory : ScriptableObject, IFactory<Foo>
    {
        public Foo Create()
        {
            // ...
            return new Foo();
        }
    }

    Container.Bind<Foo>().FromIFactory(x => x.To<FooFactory>().FromScriptableObjectResource("FooFactory")).AsSingle();
    ```

    或者，您可能希望将自定义工厂放置在这样的子容器中：

    ```csharp
    public class FooFactory : IFactory<Foo>
    {
        public Foo Create()
        {
            return new Foo();
        }
    }

    public override void InstallBindings()
    {
        Container.Bind<Foo>().FromIFactory(x => x.To<FooFactory>().FromSubContainerResolve().ByMethod(InstallFooFactory)).AsSingle();
    }

    void InstallFooFactory(DiContainer subContainer)
    {
        subContainer.Bind<FooFactory>().AsSingle();
    }
    ```

    另请注意，以下两行是等效的：

    ```csharp
    Container.Bind<Foo>().FromFactory<FooFactory>().AsSingle();
    Container.Bind<Foo>().FromIFactory(x => x.To<FooFactory>().AsCached()).AsSingle();
    ```

1. **FromComponentInNewPrefab**  - 将给定的预制件实例化为新的游戏对象，在其上注入任何MonoBehaviour，然后以类似于`GetComponentInChildren`的方式搜索结果类型**ResultType**

    ```csharp
    Container.Bind<Foo>().FromComponentInNewPrefab(somePrefab);
    ```

    **ResultType**必须是一个接口，或者在这种情况下从UnityEngine.MonoBehaviour / UnityEngine.Component派生

    请注意，如果预制件上的**ResultType**有多个匹配项，则它只会遇到遇到的第一个匹配项，就像GetComponentInChildren的工作方式一样。 因此，如果您要绑定预制件并且没有要绑定的特定MonoBehaviour / Component，则可以选择Transform，它将匹配预制件的根。

1. **FromComponentsInNewPrefab**  - 与FromComponentInNewPrefab相同，但将匹配多个值或零值。 您可以使用它作为示例，然后在某处注入`List<ContractType>`。 可以认为类似于`GetComponentsInChildren`

1. **FromComponentInNewPrefabResource**  - 将给定的预制件（在给定的资源路径中找到）实例化为新的游戏对象，在其上注入任何MonoBehaviour，然后以类似的方式搜索结果类型**ResultType**。 GetComponentInChildren`工作（因为它将返回找到的第一个匹配值）

    ```csharp
    Container.Bind<Foo>().FromComponentInNewPrefabResource("Some/Path/Foo");
    ```

    **ResultType**必须是一个接口，或者在这种情况下从UnityEngine.MonoBehaviour / UnityEngine.Component派生

1. **FromComponentsInNewPrefabResource**  - 与FromComponentInNewPrefab相同，但将匹配多个值或零值。 您可以使用它作为示例，然后在某处注入`List<ContractType>`。 可以认为类似于`GetComponentsInChildren`

1. **FromNewComponentOnNewGameObject**  - 创建一个新的空游戏对象，然后在其上实例化给定类型的新组件。

    ```csharp
    Container.Bind<Foo>().FromNewComponentOnNewGameObject();
    ```

    **ResultType**必须是一个接口，或者在这种情况下从UnityEngine.MonoBehaviour / UnityEngine.Component派生

1. **FromNewComponentOnNewPrefab**  - 将给定的预制件实例化为新的游戏对象，并在新游戏对象的根上实例化给定组件的新实例。 注意：预制件不必包含给定组件的副本。

    ```csharp
    Container.Bind<Foo>().FromNewComponentOnNewPrefab(somePrefab);
    ```

    **ResultType**必须是一个接口，或者在这种情况下从UnityEngine.MonoBehaviour / UnityEngine.Component派生

1. **FromNewComponentOnNewPrefabResource**  - 实例化给定的预制件（在给定的资源路径中找到），并在新游戏对象的根上实例化给定组件的新实例。 注意：预制件不必包含给定组件的副本。

    ```csharp
    Container.Bind<Foo>().FromNewComponentOnNewPrefabResource("Some/Path/Foo");
    ```

    **ResultType**必须是一个接口，或者在这种情况下从UnityEngine.MonoBehaviour / UnityEngine.Component派生

1. **FromNewComponentOn**  - 在给定的游戏对象上实例化给定类型的新组件

    ```csharp
    Container.Bind<Foo>().FromNewComponentOn(someGameObject);
    ```

    **ResultType**必须是一个接口，或者在这种情况下从UnityEngine.MonoBehaviour / UnityEngine.Component派生

1. **FromNewComponentSibling**  - 在当前transform上实例化给定的新组件。 这里的当前transform取自被注入的对象，因此必须是MonoBehaviour派生类型。

    ```csharp
    Container.Bind<Foo>().FromNewComponentSibling();
    ```
    请注意，如果给定的组件类型已经附加到当前transform，那么它将返回该transform而不是创建新组件。 因此，此绑定语句的功能类似于Unity的[RequireComponent]属性。

    **ResultType**必须是一个接口，或者在这种情况下从UnityEngine.MonoBehaviour / UnityEngine.Component派生

    另请注意，如果非MonoBehaviour请求给定类型，则会抛出异常，因为在这种情况下没有当前transform。

1. ** romComponentInHierarchy**  - 查找与当前上下文关联的场景层次结构中的组件，以及与任何父上下文关联的层次结构。 与`GetComponentInChildren`类似，它将返回找到的第一个匹配值

    ```csharp
    Container.Bind<Foo>().FromComponentInHierarchy().AsSingle();
    ```

    **ResultType**必须是一个接口，或者在这种情况下从UnityEngine.MonoBehaviour / UnityEngine.Component派生

    在上下文是SceneContext的最常见情况下，这将搜索整个场景层次结构（除了任何子上下文，例如GameObjectContext）。 换句话说，当前上下文是场景上下文时，它的行为类似于`GameObject.FindObjectsOfType`。 请注意，由于这可能是一个大搜索，因此应谨慎使用，就像应该谨慎使用`GameObject.FindObjectsOfType`一样。

    在上下文是GameObjectContext的情况下，它将仅搜索游戏对象根目录内部和下面（以及任何父上下文）。

    在上下文是ProjectContext的情况下，它只会在项目上下文预制中进行搜索

1. **FromComponentsInHierarchy**  - 与FromComponentInHierarchy相同，但将匹配多个值或零值。 您可以使用它作为示例，然后在某处注入`List<ContractType>`。 可以认为类似于`GetComponentsInChildren`

1. **FromComponentSibling**  - 通过搜索附加到当前变换的组件来查找给定的组件类型。 这里的当前变换取自被注入的对象，因此必须是MonoBehaviour派生类型。

    ```csharp
    Container.Bind<Foo>().FromComponentSibling();
    ```

    **ResultType**必须是一个接口，或者在这种情况下从UnityEngine.MonoBehaviour / UnityEngine.Component派生

    请注意，如果非MonoBehaviour请求给定类型，则会抛出异常，因为在这种情况下没有当前transform。

1. **FromComponentsSibling**  - 与FromComponentSibling相同，除了匹配多个值或零值。

1. **FromComponentInParents**  - 通过搜索给定组件类型的当前transform或任何父级来查找组件。 这里的当前transform取自被注入的对象，因此必须是MonoBehaviour派生类型。

    ```csharp
    Container.Bind<Foo>().FromComponentInParents();
    ```

    **ResultType**必须是一个接口，或者在这种情况下从UnityEngine.MonoBehaviour / UnityEngine.Component派生
    
    请注意，如果非MonoBehaviour请求给定类型，则会抛出异常，因为在这种情况下没有当前transform。

1. **FromComponentsInParents**  - 与FromComponentInParents相同，但会匹配多个值或零值。 您可以使用它作为示例，然后在某处注入`List<ContractType>`

1. **FromComponentInChildren**  - 通过搜索给定组件类型的当transform或任何子变换来查找组件。 这里的当前transform取自被注入的对象，因此必须是MonoBehaviour派生类型。 与`GetComponentInChildren`类似，它将返回找到的第一个匹配值

    ```csharp
    Container.Bind<Foo>().FromComponentInChildren();
    ```

    **ResultType**必须是一个接口，或者在这种情况下从UnityEngine.MonoBehaviour / UnityEngine.Component派生

    请注意，如果非MonoBehaviour请求给定类型，则会抛出异常，因为在这种情况下没有当前transform。

1. **FromComponentsInChildren**  - 与FromComponentInChildren相同，但将匹配多个值或零值。 您可以使用它作为示例，然后在某处注入`List <ContractType>`。 可以认为类似于`GetComponentsInChildren`

1. **FromNewComponentOnRoot**  - 在当前上下文的根上实例化给定组件。 这通常与GameObjectContext一起使用。

    ```csharp
    Container.Bind<Foo>().FromNewComponentOnRoot();
    ```

    **ResultType**必须是一个接口，或者在这种情况下从UnityEngine.MonoBehaviour / UnityEngine.Component派生

1. **FromResource**  - 通过调用Unity3d函数`Resources.Load`来创建**ResultType**。 这可以用于加载`Resources.Load`可以加载的任何类型，例如纹理，声音，预制件等。

    ```csharp
    Container.Bind<Texture>().WithId("Glass").FromResource("Some/Path/Glass");
    ```

1. **FromResources**  - 与FromResource相同，除了匹配多个值或零值。 您可以使用它作为示例，然后在某处注入`List<ContractType>`

1. **FromScriptableObjectResource**  - 直接绑定到给定资源路径上的可脚本化对象实例。 注意：在Unity编辑器内部对此值的更改将被永久保存。如果这是不合需要的，请使用FromNewScriptableObjectResource。

    ```csharp
    public class Foo : ScriptableObject
    {
    }

    Container.Bind<Foo>().FromScriptableObjectResource("Some/Path/Foo");
    ```

1. **FromNewScriptableObjectResource**  - 与FromScriptableObjectResource相同，但它将实例化给定脚本对象资源的新副本。 如果您希望拥有给定脚本化对象资源的多个不同实例，或者您希望确保在运行时更改不会影响脚本化对象的已保存值，则此选项非常有用。

1. **FromResolve**  - 通过对容器执行另一次查找来获取实例（换句话说，调用`DiContainer.Resolve<ResultType>()`）。 请注意，要使其工作，**ResultType**必须绑定在单独的绑定语句中。 当您想要将接口绑定到另一个接口时，此构造方法特别有用，如下面的示例所示

    ```csharp
    public interface IFoo
    {
    }

    public interface IBar : IFoo
    {
    }

    public class Foo : IBar
    {
    }

    Container.Bind<IFoo>().To<IBar>().FromResolve();
    Container.Bind<IBar>().To<Foo>();
    ```
1. **FromResolveAll**  - 与FromResolve相同，但将匹配多个值或零值。

1. **FromResolveGetter<ObjectType>**  - 从另一个依赖项的属性中获取实例，该依赖项是通过对容器执行另一次查找获得的（换句话说，调用`DiContainer.Resolve<ObjectType>()`然后访问一个值 在返回的类型**ResultType**）的实例上。 请注意，要使其工作，**ObjectType**必须绑定在单独的绑定语句中。

    ```csharp
    public class Bar
    {
    }

    public class Foo
    {
        public Bar GetBar()
        {
            return new Bar();
        }
    }

    Container.Bind<Foo>();
    Container.Bind<Bar>().FromResolveGetter<Foo>(x => x.GetBar());
    ```

1. **FromResolveAllGetter<ObjectType>**  - 与FromResolveGetter相同，但将匹配多个值或零值。

1. **FromSubContainerResolve** - 通过对子容器进行查找来获取ResultType。 请注意，为此，子容器必须具有ResultType的绑定。 这种方法非常强大，因为它允许您将相关的依赖项组合在一个迷你容器中，然后只暴露某些类（也称为<a href="https://en.wikipedia.org/wiki/Facade_pattern">"Facades"</a>）以在更高级别对这组依赖项进行操作。 有关使用子容器的更多详细信息，请参阅[本节](sub-containers-and-facades.md)。 有几种方法可以定义子容器：

    1.** ByNewPrefabMethod**  - 通过实例化一个新的预制件来初始化子容器。 请注意，与`ByNewContextPrefab`不同，此绑定方法不要求在预制件上附加GameObjectContext。 在这种情况下，GameObjectContext是动态添加的，然后使用给定的安装程序方法运行。

        ```csharp
        Container.Bind<Foo>().FromSubContainerResolve().ByNewPrefabMethod(MyPrefab, InstallFoo);

        void InstallFoo(DiContainer subContainer)
        {
            subContainer.Bind<Foo>();
        }
        ```
    1. **ByNewPrefabInstaller**  - 通过实例化新的预制件来初始化子容器。 与ByNewPrefabMethod相同，除了它使用给定的安装程序而不是方法初始化动态创建的GameObjectContext。
    
        ```csharp
        Container.Bind<Foo>().FromSubContainerResolve().ByNewPrefabInstaller<FooInstaller>(MyPrefab);

        class FooInstaller : Installer
        {
            public override void InstallBindings()
            {
                Container.Bind<Foo>();
            }
        }
        ```

    1. **ByNewPrefabResourceMethod**  - 初始化子容器实例化通过`Resources.Load`获得的新预制件。 请注意，与`ByNewPrefabResource`不同，此绑定方法不要求将预定的GameObjectContext附加到预制件上。 在这种情况下，GameObjectContext是动态添加的，然后使用给定的安装程序方法运行。
        
        ```csharp
        Container.Bind<Foo>().FromSubContainerResolve().ByNewPrefabResourceMethod("Path/To/MyPrefab", InstallFoo);

        void InstallFoo(DiContainer subContainer)
        {
            subContainer.Bind<Foo>();
        }
        ```

    1. **ByNewPrefabResourceInstaller**  - 初始化子容器实例化通过`Resources.Load`获得的新预制件。 与ByNewPrefabResourceMethod相同，除了它使用给定的安装程序而不是方法初始化动态创建的GameObjectContext。

        ```csharp
        Container.Bind<Foo>().FromSubContainerResolve().ByNewPrefabResourceInstaller<FooInstaller>("Path/To/MyPrefab");

        class FooInstaller : MonoInstaller
        {
            public override void InstallBindings()
            {
                Container.Bind<Foo>();
            }
        }
        ```

    1. **ByNewGameObjectInstaller**  - 通过实例化一个空的游戏对象，附加GameObjectContext，然后使用给定的安装程序进行安装来初始化子容器。 这种方法与ByInstaller非常相似，但具有以下优点：

        - 将正确调用任何ITickable，IInitializable，IDisposable等绑定
        - 在子容器内实例化的任何新游戏对象都将成为游戏对象上下文对象下的父级
        - 您可以通过破坏游戏对象上下文游戏对象来销毁子容器

    1. **ByNewGameObjectMethod**  - 与ByNewGameObjectInstaller相同，但子容器是基于给定方法而不是安装程序类型初始化的。

    1. **ByMethod**  - 使用方法初始化子容器。

        ```csharp
        Container.Bind<Foo>().FromSubContainerResolve().ByMethod(InstallFooFacade);

        void InstallFooFacade(DiContainer subContainer)
        {
            subContainer.Bind<Foo>();
        }
        ```

        请注意，在使用ByMethod时，如果要在子容器内使用诸如ITickable，IInitializable，IDisposable等zenject接口，那么您还必须使用WithKernel绑定方法，如下所示：

        ```csharp
        Container.Bind<Foo>().FromSubContainerResolve().ByMethod(InstallFooFacade).WithKernel();

        void InstallFooFacade(DiContainer subContainer)
        {
            subContainer.Bind<Foo>();
            subContainer.Bind<ITickable>().To<Bar>();
        }
        ```
    1. **ByInstaller**  - 使用从`Installer`派生的类初始化子容器。 与`ByMethod`相比，这可能是一种更清晰，更不容易出错的替代方案，特别是如果您需要将数据注入安装程序本身。 不易出错，因为在使用ByMethod时，通常会在方法中意外使用Container而不是subContainer。

        ```csharp
        Container.Bind<Foo>().FromSubContainerResolve().ByInstaller<FooFacadeInstaller>();

        class FooFacadeInstaller : Installer
        {
            public override void InstallBindings()
            {
                Container.Bind<Foo>();
            }
        }
        ```

        请注意，在使用ByInstaller时，如果要在子容器内使用诸如ITickable，IInitializable，IDisposable等zenject接口，那么您还必须使用WithKernel绑定方法，如下所示：

        ```csharp
        Container.Bind<Foo>().FromSubContainerResolve().ByInstaller<FooFacadeInstaller>().WithKernel();
        ```

    1. **ByNewContextPrefab**  - 通过实例化新的预制件来初始化子容器。 请注意，预制件必须包含附加到根游戏对象的`GameObjectContext`组件。 有关`GameObjectContext`的详细信息，请参阅[此部分](sub-containers-and-facades.md)


        ```csharp
        Container.Bind<Foo>().FromSubContainerResolve().ByNewContextPrefab(MyPrefab);

        // Assuming here that this installer is added to the GameObjectContext at the root
        // of the prefab.  You could also use a ZenjectBinding in the case where Foo is a MonoBehaviour
        class FooFacadeInstaller : MonoInstaller
        {
            public override void InstallBindings()
            {
                Container.Bind<Foo>();
            }
        }
        ```

    1. **ByNewContextPrefabResource**  - 初始化子容器实例化通过`Resources.Load`获得的新预制件。 请注意，预制件必须包含附加到根游戏对象的`GameObjectContext`组件。

        ```csharp
        Container.Bind<Foo>().FromSubContainerResolve().ByNewContextPrefabResource("Path/To/MyPrefab");
        ```

    1. **ByInstance**  - 直接使用您自己提供的给定DiContainer实例初始化子容器。 这仅适用于一些罕见的边缘情况。

    1. **FromSubContainerResolveAll**  - 与FromSubContainerResolve相同，除了匹配多个值或零值。