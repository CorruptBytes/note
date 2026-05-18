概述

# 项目模板

> 所有Fabric模组项目均有一个固定的模板，有多种方法可以获得模板项目并以此为基础编写自己的项目

## 获得模板

**方式一**

1. 在官方github仓库中拉取项目模板，排除仓库中的`LICENSE`和`README.md`文件

   ```http
   https://github.com/FabricMC/fabric-example-mod/
   ```

2. 删除`.git`相关版本管理文件夹

**方式二: Fabric 模板模组生成器**

1. 使用官方提供的模板生成工具,填写必要字段，如模组名称，包名，Minecraft版本等

   ```http
   https://fabricmc.net/develop/template/
   ```

   - 包名应该是全小写字母，由点隔开；命名必须唯一，以避免和其他开发者的包冲突，可以命名为`com.yourname.mod-id`

**方式三:Idea插件`Minecraft Development`**

## 修改模板

> 如果模板是复制而来，需要对模板中文件进行一定修改

- 修改项目的 `gradle.properties` 文件，把 `maven_group` 和 `archive_base_name` 修改为匹配你的模组的信息。
- 修改项目中的 `fabric.mod.json` 文件，把 `id`、`name` 和 `description` 修改为匹配你的模组的信息。
- 确保更新 Minecraft、映射、Fabric Loader 和 Fabric Loom 的版本——所有这些都可以通过 https://fabricmc.net/develop/ 查询，以匹配你希望的目标版本。

## 加速Fabric模组构建

### **配置代理**

> 可以设置全局代理(作用于所有Gradle项目)或项目代理(作用于当前项目)

```
systemProp.http.proxyHost=代理地址或域名
systemProp.http.proxyPort=代理端口
systemProp.https.nonProxyHosts=10.*|localhost

systemProp.https.proxyHost=代理地址或域名
systemProp.https.proxyPort=代理端口
systemProp.https.nonProxyHosts=10.*|localhost
```

- 项目代理配置在前的 Fabric 模组文件夹之下的` gradle.properties`
- 全局代理配置在当前用户目录下的`.gradle\gradle.properties`中
- 为保证gradle正常工作，需要同时配置`https`以及`http`代理

### 换源

> 切换`Fabric`以及 `MavenCenter`源

1. 将`settings.gradle`替换为以下内容，加速 fabric-loom 的下载

   ```groovy
   pluginManagement {
       repositories {
           maven {
               name = 'Fabric'
               url = 'https://repository.hanbings.io/proxy'
           }
           gradlePluginPortal()
       }
   }
   ```

2. 在`build.gradle`中的`repositories`节点修改为以下内容，用于加速 FabricAPI 的下载

   ```groovy
   repositories {
       maven {
           url 'https://maven.aliyun.com/nexus/content/groups/public'
       }
       maven {
           url 'https://repository.hanbings.io/proxy'
       }
   }
   ```

# 模组初始化器

# 数据生成

> 数据生成是 Fabric API 中的新模块，允许动态生成配方、语言文件、战利品表、进度以及几乎所有带有自定义提供器的一切

- 每次修改生成数据的代码后，都需要运行 gradle 命令 `**runDatagen**`。
- 生成的文件默认将创建在 `src/main/generated` 中。
- 数据生成的所有类均应写在`client`端，否则构建时可能出现问题

## 启用数据生成

**方式一**

- 在使用`Fabric`模板生成器生成项目模板时，勾选 `**Data generation**` 框,模板生成器会自动创建`runDatagen`命令,并提供直接可运行的配置

**方式二**

1. 修改`build.gradle`文件，添加下列配置

   ```groovy
   //
   // ... （文件剩余部分）
   //1.21.4前
   
   fabricApi {
       configureDataGeneration()
   }
   
   //1.21.4以上
   fabricApi {
       configureDataGeneration() {
           client = true
       }
   }
   ```

2. 创建一个新类`ExampleModDataGenerator`,实现`DataGeneratorEntrypoint`接口,并实现接口中的`onInitializeDataGenerator`抽象方法

   ```java
   public class ExampleModDataGenerator implements DataGeneratorEntrypoint {
    
       @Override
       public void onInitializeDataGenerator(FabricDataGenerator generator) {
           //创建包,生成的数据将放入此包中
           FabricDataGenerator.Pack pack = generator.createPack();
       }
    
   }
   ```

   - `runDatagen`命令运行时，会调用`onInitializeDataGenerator`方法

3. 在`fabric.mod.json`文件中添加配置，告诉`Fabric`入口点

   ```json
   {
    
     // ...（文件的剩余部分）
    
     "entrypoints": {
       "fabric-datagen": [
         "com.example.ExampleModDataGenerator"
       ],
       "main": [
         "com.example.ExampleMod"
       ],
       "client": [
         "com.example.ExampleModClient"
       ]
     },
    
    
     // ...（文件的剩余部分）
    
   }
   ```

**配置重复文件生成策略**

> 在`build.gradle`中进行配置

```groovy
processResources {
	duplicatesStrategy = DuplicatesStrategy.WARN
}
```

- 值为一个枚举值

|              |                                                              |      |
| ------------ | ------------------------------------------------------------ | ---- |
| `EXCLUDE`    | 通过忽略在同一路径上创建的后续项，不允许重复。               |      |
| `FAIL`(默认) | 当要在同一路径上创建后续项时，引发 `DuplicateFileCopyingException`。 |      |
| `INCLUDE`    | 不要试图防止重复。                                           |      |
| `INHERIT`    | 使用与父副本规范相同的策略。                                 |      |
| `WARN`       | 不要尝试防止重复，而是在多个项目将在同一路径上创建时记录警告消息。 |      |



## 提供器

> FabricAPI提供了各种提供器,用于自动生成进度，战利品，标签，配方，语言文件等内容。通过继承已有的提供器获得所需的提供器

- 可以将生成器类放到单独文件中，不过更建议将生成器作为入口类的静态内部类

1. 创建新类并继承`Fabric API`提供的提供器父类，不同生成内容继承不同的生成器

   ```java
    private static class MyTagGenerator extends FabricTagProvider.ItemTagProvider {
   
           
           public MyTagGenerator(FabricDataOutput output, CompletableFuture<RegistryWrapper.WrapperLookup> registriesFuture) {
               super(output, registriesFuture);
           }
   
           @Override
           protected void configure(RegistryWrapper.WrapperLookup wrapperLookup) {
   
           }
       }
   ```

2. 在入口类注册提供器

   ```
   pack.addProvider(MyTagGenerator::new);
   ```

3. 重写提供器的方法进行数据生成

   ```java
          @Override
           protected void configure(RegistryWrapper.WrapperLookup wrapperLookup) {
   			valueLookupBuilder(BlockTags.PICKAXE_MINEABLE)
           }
   ```

   

### 标签提供器

> `FabricAPI`提供了`FabricTagBuilder`及其子类作为不同类型标签的提供器

```java
 private static class MyTagGenerator extends FabricTagProvider.ItemTagProvider {

        
        public MyTagGenerator(FabricDataOutput output, CompletableFuture<RegistryWrapper.WrapperLookup> registriesFuture) {
            super(output, registriesFuture);
        }

        @Override
     	//configure方法用于生成文件,生成文件前需要先创建标签文件的构建器
        protected void configure(RegistryWrapper.WrapperLookup wrapperLookup) {
			valueLookupBuilder(BlcokTags.).add();
        }
    }
```

- 通常不直接继承`FabricTagProvider<T>`,而是继承`FabricTagProvider<T>`的内部子类,其内部子类已经配置了各类标签的生成前缀。
- 泛型表示提供标签的类型

#### `FabricTagBuilder`

> 不同类型标签提供器的父类,提供了所有标签生成器的共同方法

![image-20251016113303822](D:\笔记\图片\image-20251016113303822.png)

- 所有子类均为`FabricTagBuilder`的内部子类,最下层子类仅为其父类指定泛型并提供默认值构造方法，并没有提供额外功能。

**`FabricValueLookupTagProvider`**

> `FabricTagBuilder`的内部子类,支持通过添加注册实例的方式添加标签

|                                                              |                                |      |
| ------------------------------------------------------------ | ------------------------------ | ---- |
| `ProvidedTagBuilder<T, T> valueLookupBuilder(TagKey<T> tag)` | 根据`TagKey`生成一个标签构建器 |      |

**`ProvidedTagBuilder`**

> 通过调用其中方法向标签中添加内容,支持链式编程

|                              |                            |      |
| :--------------------------: | -------------------------- | ---- |
| `forceAddTag(TagKey<T> tag)` | 向标签中添加其他标签元素   |      |
|      `add(E... values)       | 根据元素实例向标签添加元素 |      |

**`BlockTags`**

> 将标识原版常用的方块标签路径的`TagKey`封装为静态字段，供使用者直接引用

### 模型提供器

```java
public class FabricDocsReferenceModelProvider extends FabricModelProvider {
	public FabricDocsReferenceModelProvider(FabricDataOutput output) {
		super(output);
	}
        //生成方块状态文件与模型文件
        @Override
        public void generateBlockStateModels(BlockStateModelGenerator blockStateModelGenerator) {
            //参数blockStateModelGenerator负责生成所有的JSON文件
        }

	//生成物品相关模型文件
	@Override
	public void generateItemModels(ItemModelGenerator itemModelGenerator) {
        //itemModelGenerator负责生成JSON文件

	}

	@Override
	public String getName() {
		return "FabricDocsReference Model Provider";
	}
}
```

#### 方块模型

- 由`generateBlockStateModels`方法生成
- 方法中的`BlockStateModelGenerator`对象负责生成JSON文件
- 该方法不但生成方块模型文件，还会根据方块的状态属性生成对应的方块状态文件以及模型映射文件

```java
@Override
public void generateBlockStateModels(BlockStateModelGenerator blockStateModelGenerator) {
}
```

**Cube All**

```java
blockStateModelGenerator.registerSimpleCubeAll(ModBlocks.STEEL_BLOCK);
```

- 通过`registerSimpleCubeAll`方法生成简单的`cube_all`模型方块，所有六个面都使用一个纹理

**其他模型**

```java
blockStateModelGenerator.registerSingleton(ModBlocks.PIPE_BLOCK, TexturedModel.CUBE_ALL);
```

- `TexturedModel` 中以静态常量的方式存储了所有父类模型，用于为方块指定特殊的父类模型。

#### 物品模型

- 物品模型由`generateItemModels`方法生成
- `itemModelGenerator`对象用于生成模型，不但会生成模型还会生成对应的模型映射文件

**简单物品模型**

```java
itemModelGenerator.register(TutorialItems.CUSTOM_ITEM, Models.GENERATED);
```

- 所有父类模型作为`Models`的静态字段被封装在一起

|                                         |                                              |      |
| :-------------------------------------: | -------------------------------------------- | ---- |
| `void register(Item item, Model model)` | 生成指定父类模型的模型文件与物品模型映射文件 |      |
|        `void register(Item item`        | 只生成物品模型映射文件                       |      |
|                                         |                                              |      |

### 翻译键提供器

```java
public static class MyTranslationProvider extends FabricLanguageProvider {

		protected MyTranslationProvider(FabricDataOutput dataOutput, CompletableFuture<RegistryWrapper.WrapperLookup> registryLookup) {
			super(dataOutput, registryLookup);
		}

		@Override
		public void generateTranslations(RegistryWrapper.WrapperLookup registryLookup, TranslationBuilder translationBuilder) {
			translationBuilder.add(ModItems.DIAMOND_PROSPECTOR,"钻石探测仪");
		}
	}
```

### 配方提供器

```java
	public static class ModRecipeProvider extends FabricRecipeProvider {

		public ModRecipeProvider(FabricDataOutput output, CompletableFuture<RegistryWrapper.WrapperLookup> registriesFuture) {
			super(output, registriesFuture);
		}

		@Override
		protected RecipeGenerator getRecipeGenerator(RegistryWrapper.WrapperLookup registryLookup, RecipeExporter exporter) {
			//Json文件由RecipeGenerator对象生成
			return new RecipeGenerator(registryLookup,exporter) {
				@Override
				public void generate() {
					createShapeless(RecipeCategory.FOOD,ModItems.DIAMOND_PROSPECTOR)
							.input(ModItems.ANTHRACITE_COAL)
							.criterion(hasItem(ModItems.DIAMOND_PROSPECTOR), conditionsFromItem(ModItems.ANTHRACITE_COAL))
							.offerTo(exporter);
				}
			};
		}

		@Override
		public String getName() {
			return "";
		}
	}
```

- `getRecipeGenerator`中返回一个`RecipeGenerator`对象，通常采用匿名内部类的形式并实现`generate`方法

#### `RecipeGenerator`

> 封装了各种方法，用于获得不同类型配方的`Builder`，通过`Builder`设置配方内容最后通过`offerTo`方法将配方交给`exporter`。

**无序配方**

- 通过`createShapeless`方法生成无序配方构建器。

```java
createShapeless(RecipeCategory.FOOD,ModItems.DIAMOND_PROSPECTOR)
							.input(ModItems.ANTHRACITE_COAL)
							.criterion(hasItem(ModItems.DIAMOND_PROSPECTOR), conditionsFromItem(ModItems.ANTHRACITE_COAL))
							.offerTo(exporter);
```

| 方法名                                                 | 功能                                             | 说明                                                       |
| ------------------------------------------------------ | ------------------------------------------------ | ---------------------------------------------------------- |
| `input(ItemConvertible item)`                          | 添加配方所需材料                                 | 可多次调用以添加多个输入材料（位置无关）                   |
| `criterion(String name, CriterionCondition condition)` | 定义配方解锁条件（advancement 触发）             | 用于让配方在玩家获得特定物品后显示在工作台配方书中         |
| `offerTo(Consumer<RecipeJsonProvider> exporter)`       | 将配方内容注册到数据导出器                       | 交给 `DataGenerator` 生成对应的 `recipes/<name>.json` 文件 |
| `conditionsFromItem(ModItems.ANTHRACITE_COAL)`         | 表示「当玩家拥有这个物品时，触发该条件」         |                                                            |
| `hasItem(ModItems.DIAMOND_PROSPECTOR)`                 | 返回一个条件名称（字符串），表示这个条件的标识符 |                                                            |
|                                                        |                                                  |                                                            |

### 战利品表提供器

#### 方块战利品表

```java
public class ExampleModBlockLootTableProvider extends FabricBlockLootTableProvider {
	protected ExampleModBlockLootTableProvider(FabricDataOutput dataOutput, CompletableFuture<RegistryWrapper.WrapperLookup> registryLookup) {
		super(dataOutput, registryLookup);
	}

	@Override
	public void generate() {
        //基础作物方块掉落
        addDrop(ModBlocks.STRAWBERRY, 
                cropDrops(ModBlocks.STRAWBERRY, ModItems.STRAWBERRY,ModItems.STRAWBERRY_SEEDS, BlockStatePropertyLootCondition.builder(ModBlocks.STRAWBERRY)            .properties(StatePredicate.Builder.create().exactMatch(StrawberryCropBlock.AGE, 5))));
	}
}
```

##### **`FabricBlockLootTableProvider`**

> 方块战利品提供器的父类，专门用于提供方块相关的战利品表配置文件

**战利品表添加方法**

|                             |                                                  |      |
| :-------------------------: | ------------------------------------------------ | ---- |
| `void addDrop(Block block)` | 添加一个最简单的战利品表，即挖掘掉落一个方块本体 |      |
|                             |                                                  |      |
|                             |                                                  |      |





# 物品

## 物品相关类

### `Item`

> MC中所有物品的基类，代表一种物品。

**成员方法**

|                                                              |                                                              |      |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ---- |
| `void inventoryTick(ItemStack stack, ServerWorld world, Entity entity, @Nullable EquipmentSlot slot)` | 如果物品在玩家或其他实体的库存中，则每游戏刻在服务器上调用一次 |      |
|                                                              |                                                              |      |
|                                                              |                                                              |      |
|                                                              |                                                              |      |



### `ItemStack`

> 意为物品堆，表示堆叠的多个物品。在MC中，每一个物品格子(slot)中都不是直接保存`Item`对象，而是保存一个`ItemStack`对象

**常用属性**

|       属性        |                          说明                           |
| :---------------: | :-----------------------------------------------------: |
|    `Item item`    |       表示物品的类型，比如 `Items.DIAMOND_SWORD`        |
|    `int count`    |        当前堆叠数量，比如 64 表示一堆 64 个物品         |
| `NbtCompound nbt` |   附加数据，例如自定义名称、附魔、耐久、属性修饰符等    |
|  `DataComponent`  | 数据组件，在 1.20+ 后代替部分 NBT，用于统一描述物品属性 |
|  `boolean empty`  |       判断这个物品堆是否为空 (`ItemStack.EMPTY`)        |

**常用方法**

|          |      |      |
| :------: | ---- | ---- |
| `damage` |      |      |
|          |      |      |
|          |      |      |
|          |      |      |



## 自定义物品

> 在Minecraft中，添加物品的关键就是注册物品；注册物品后，就可以通过`give`指令在游戏中获得物品

**1.21.2前**

```java
public final class TutorialItems {
 
    private TutorialItems() {}
 
    // 新物品的实例
    public static final Item CUSTOM_ITEM = register("custom_item", new Item(new Item.Settings()));
 
    public static <T extends Item> T register(String path, T item) {
   
        return Registry.register(Registries.ITEM, Identifier.of("tutorial", path), item);
    }
 
    public static void initialize() {
    }
}
```

**1.21.2+**

> 1.21.2后，物品注册重写了，需要把 `RegistryKey` 存储到 `Item.Settings` 中，这样模型和翻译键才会正确地存储在物品组件中

```java
public final class TutorialItems {
  private TutorialItems() {
  }
 
  public static final Item CUSTOM_ITEM = register("custom_item", Item::new, new Item.Settings());
 
  public static Item register(String path, Function<Item.Settings, Item> factory, Item.Settings settings) {
    final RegistryKey<Item> registryKey = RegistryKey.of(RegistryKeys.ITEM, Identifier.of("tutorial", path));
    return Items.register(registryKey, factory, settings);
  }
 
  public static void initialize() {
  }
}
```

- `Items`是原版专门用于注册物品的类，`Items.register` 中，注册表键registry key会先写到 `settings` 中，然后使用那个 `settings` 去创建物品。
- 也可以使用`Registry.register()`,这是是最底层的注册类，如果在`1.21.2`以上版本使用它，需要注意要将注册表键registry key写入`settings`后再注册物品。

- 一般会封装一个专门注册物品的类，将所有需要注册的物品封装为类的静态字段。

- 所有注册方法都会返回一个物品实例

- 为使JVM加载此类，需要在模组初始化过程中引用模组的一些方法或字段

  ```java
  public class ExampleMod implements ModInitializer {
      @Override
      public void onInitialize() {
          TutorialItems.initialize();
      }
  }
  ```

## 物品相关资源文件

> 一个物品的完整资源包括纹理，模型，模型映射与命名翻译键。

- 物品模型：`./resources/assets/<namespace>/models/item/<custom_item>.json`
- 物品纹理：`…/resources/assets/<namespace>/textures/item/<custom_item>.png`
- 物品模型映射（自从 1.21.4）：`…/resources/assets/<namespace>/items/<custom_item>.json`
- 翻译键:`./resources/assets/mod-id/lang/<language>.json`,可以为不同语言提供不同的翻译键,中文为`zh_cn`,英文为`en_us`

### **纹理**

> 定义物品或方块的**外观颜色、贴图**，必须是一个PNG格式的图片

- 技术上图片可以任意命名，但是一般图片命名与物品的命名保持一致

### **模型**

> 定义物品的渲染方式,并指定物品贴图

- 在1.21.4前，MC会将物品注册时的标识符映射为资源路径并自动绑定物品与模型,因此要保证模型文件与物品同名。
- 在1.21.4后，物品渲染的模型在物品模型映射文件中定义

```json
{
  "parent": "item/generated",
  "textures": {
    "layer0": "<namespace>:item/<img_name>"
  }
}
```

-  `parent`结点配置模型要继承的模型。用于改变物品在手中以及在物品栏内等情形下的渲染，`item/generated` 用于许多简单的物品。`item/handheld` 用于手持其纹理左下角的物品。
- `textures/layer0` 节点通过纹理的标识符指定纹理位置，标识符会被自动映射为纹理的路径。

### **物品模型映射**

> 物品模型渲染从1.21.4版本添加,专门用来指定物品渲染所使用的模型

- MC会将物品注册时的标识符映射为资源路径并自动绑定物品与物品模型映射

```json
{
  "model": {
    "type": "model",
    "model": "<namespace>:item/<custom_item>"
  }
}
```

- `type`：这是模型的类型。 对于大多数物品，应该为 `minecraft:model`
- `model`：这是模型的标识符，对应一个`./resources/assets/<namespace>/models/item` 中的模型，格式为：`<mod-id>:item/<item_name>`

### **命名翻译键**

```json
{
  "item.<mod_id>.<item_name>": "Suspicious Substance"
}
```

- 上述翻译键会由MC自动映射为物品的命名。

## 物品行为

> **想要修改物品的行为，可以继承`Item`物品类并重写`Item`中物品的默认行为，在注册时使用自定义物品对象代替父类物品对象即可**

**常用方法**

|               |                      |      |
| :-----------: | :------------------: | ---- |
|     `use`     |   物品被使用时调用   |      |
| `useOnBlock`  | 物品对方块使用时调用 |      |
| `useOnEntity` | 物品向实体使用时调用 |      |
|               |                      |      |

**添加物品提示信息**

- 重写`Item`类的`appendTooltip`方法

```java
//对于 1.18.2 及之前的版本：
@Override
public void appendTooltip(ItemStack itemStack, World world, List<Text> tooltip, TooltipContext tooltipContext) {
 
    // 默认为白色文本
    tooltip.add(new TranslatableText("item.tutorial.custom_item.tooltip"));
 
    // 格式化为红色文本
    tooltip.add(new TranslatableText("item.tutorial.custom_item.tooltip").formatted(Formatting.RED) );
}
//对于 1.19 之后的版本：
@Override
public void appendTooltip(ItemStack itemStack, World world, List<Text> tooltip, TooltipContext tooltipContext) {
    tooltip.add(Text.translatable("item.tutorial.custom_item.tooltip"));
}
//对于 1.20.5 之后的版本：
@Override
public void appendTooltip(ItemStack itemStack, TooltipContext context, List<Text> tooltip, TooltipType type) {
    tooltip.add(Text.translatable("item.tutorial.custom_item.tooltip"));
}
//从1.21.5开始，物品提示改为使用物品组件实现，上述方法被废弃，可以借助 Fabric API 添加自定义的物品提示，在代码的模组初始化器部分加入以下代码添加自定义物品提示。(但是仍然可以通过重写appendTooltip实现添加提示)
   ItemTooltipCallback.EVENT.register((itemStack, tooltipContext, tooltipType, list) -> {
      if (!itemStack.isOf(TutorialItems.CUSTOM_ITEM)) {
        return;
      }
      list.add(Text.translatable("item.tutorial.custom_item.tooltip"));
    });
```

- 每次调用都会添加一行提示

## 物品属性

**大部分的物品属性可以通过物品所持有的`Item.Settings`对象进行设置**

- 不可破坏

  ```java
  	// 对于 1.21.2 之前的版本：
  	new Item.Settings()
      .component(DataComponentTypes.UNBREAKABLE, new UnbreakableComponent(true))
      // 对于从 1.21.2 及以后、1.21.4 之前的版本：
          new Item.Settings()
          .component(DataComponentTypes.UNBREAKABLE, new UnbreakableComponent(true))
      // 对于从 1.21.4 及以后:
           new Item.Settings()
          .component(DataComponentTypes.UNBREAKABLE, Unit.INSTANCE)
  ```

  - 不可破坏物品的堆叠属性无效，只能为1
  

**最大堆叠数**

  ```java
  new Item.Settings().maxCount(16)
  ```

**最大耐久度**

```
maxDamage(10)
```


**物品可作燃烧物**

> 原版中通过`FuelRegistry`类中的Map注册所有燃烧物,但没有提供自定义添加燃烧物的接口，可以通过Mixin修改原版代码

- FabricAPI已经提供了封装好的接口,底层使用Mixin

  ```java

  // 对于 1.21.2 之后的版本
  FuelRegistryEvents.BUILD.register((builder, context) -> {
          	// 第一个参数为添加的物品，第二个参数为可燃烧时间
              builder.add(CUSTOM_ITEM, 300);
          });
  ```

**物品可作堆肥**

```java
 CompostingChanceRegistry.INSTANCE.add(ModItems.MY_ITEM, 0.3f);
```

**不可被火焰或岩浆销毁**

```java
new Item.Settings().fireproof()
```

**设置`item`作为翻译键前缀**

```
  new Item.Settings().maxCount(16).useItemPrefixedTranslationKey
```

- `Item`类默认翻译键前缀即为`item`。

**设置`block`作为翻译键前缀**

```
  new Item.Settings().maxCount(16).useBlockPrefixedTranslationKey
```

- 常用于注册方块物品时，将方块物品的翻译键前缀设置为`block`标识其为一个方块物品

## 合成配方

> 在1.21之前，配方位于资源目录下的`resources/data/<namespace>/recipes/`;1.21之后,位于资源目录下的`resources/data/<namespace>/recipe/`。配方为一个`json`文件，名称可以是任意名称(一般直接使用物品名)

```json 
{
  "type": "minecraft:crafting_shaped",
  "pattern": [
    "WWW",
    "WR ",
    "WWW"
  ],
  "key": {
    "W": {
      "tag": "minecraft:logs"
    },
    "R": {
      "item": "minecraft:redstone"
    }
  },
  "result": {
    "id": "tutorial:custom_item", #在1.21之前,id字段为item
    "count": 4
  }
}
```

- **type**：这是个有序合成配方。
- **result**：这是合成4个 `tutorial:custom_item` 的配方。`count` 字段是可选的，如果不指定 `count`，则默认为 1。
- **pattern**：代表合成配方的图案。每个字母代表一个物品。空格表示该槽位没有物品。每个字母代表的物品在 **key** 中定义。
- **key**：每个字母代表的物品。`W` 代表带有 `minecraft:logs` 标签的物品（即所有原木）。`R` 代表红石。关于标签的更多信息
- **category**：这个配方在解锁后在配方书中显示的分类。

## 自定义盔甲

1. 首先创建盔甲材料对象`ArmorMaterial`,定义了一套盔甲的共同属性

   ```java
   public class ModArmorMaterials {
    public static final RegistryKey<EquipmentAsset> MY_MATERIAL_KEY =RegistryKey.of(EquipmentAssetKeys.REGISTRY_KEY,Identifier.of(My_mod.MOD_ID,"ice_ether"));), Identifier.of(My_mod.MOD_ID,"material"));
       public static final ArmorMaterial MY_ARMOR_MATERIAL = new ArmorMaterial(5, Maps.newEnumMap(
               Map.of(
                       EquipmentType.BOOTS,
                       1,
                       EquipmentType.LEGGINGS,
                       1,
                       EquipmentType.CHESTPLATE,
                       1,
                       EquipmentType.HELMET,
                       1,
                       EquipmentType.BODY,
                       1
               )
       ),2, SoundEvents.ITEM_ARMOR_EQUIP_CHAIN,1,1,null,MY_MATERIAL_KEY);
   }
   ```
   
   | 参数                  | 描述                                                         |
   | :-------------------- | :----------------------------------------------------------- |
   | `durability`          | 所有盔甲的基础耐久度，用于计算使用该材质的每个盔甲的总耐久度。 这应该是你之前创建的基础耐久度常量。 |
   | `defense`             | `EquipmentType`（代表每个护甲槽位的枚举）到整数值的映射，表示材质在相应护甲槽位中使用时的防御值。 |
   | `enchantmentValue`    | 使用此材料制作的盔甲的“可附魔性”。                           |
   | `equipSound`          | 当你装备使用该材质的盔甲时播放的声音事件的注册表项。 有关声音的更多信息，请参阅[自定义声音](https://docs.fabricmc.net/zh_cn/develop/sounds/custom)页面。 |
   | `toughness`           | 一个浮点值，代表盔甲材质的“韧性”属性——本质上代表盔甲吸收伤害的能力。 |
   | `knockbackResistance` | 一个浮点值，代表盔甲材质赋予穿戴者的击退抗性。               |
   | `repairIngredient`    | 一个物品标签，代表所有能够用在铁砧中修复此材质的盔甲物品的物品。 |
| `assetId`             | 一个 注册表项，这应该是你之前创建的设备资产注册表项常量。`EquipmentAsset` |
   
2. 注册盔甲的各个部件

   ```java
   public static final Item LEATHER_HELMET = register("leather_helmet", new Item.Settings().armor(ArmorMaterials.LEATHER, EquipmentType.HELMET));
   public static final Item LEATHER_CHESTPLATE = register("leather_chestplate", new Item.Settings().armor(ArmorMaterials.LEATHER, EquipmentType.CHESTPLATE));
   public static final Item LEATHER_LEGGINGS = register("leather_leggings", new Item.Settings().armor(ArmorMaterials.LEATHER, EquipmentType.LEGGINGS));
   public static final Item LEATHER_BOOTS = register("leather_boots", new Item.Settings().armor(ArmorMaterials.LEATHER, EquipmentType.BOOTS));
   ```
   
3. 添加盔甲的纹理文件，盔甲的纹理有两种。

   - 物品在物品栏，手中以及掉落物的材质，添加方法同普通物品
   - 物品穿戴在身上的纹理，有两层
     - `assets/mod-id/textures/entity/equipment/humanoid/guidite.png` ,包含了上身和靴子。
     - `assets/mod-id/textures/entity/equipment/humanoid_leggings/guidite.png` ,包含了护腿纹理。

4. 添加关联的装备模型定义，位于`/assets/mod-id/equipment/<ID>.json`,盔甲材料的注册键ID要与json文件一致

   ```json
   {
     "layers": {
       "humanoid": [
         {
           "texture": "fabric-docs-reference:guidite"
         }
       ],
       "humanoid_leggings": [
         {
           "texture": "fabric-docs-reference:guidite"
         }
       ]
     }
   }
   ```

## 自定义食物

> 通过为`Item.Settings`实例为物品添加食物属性

|                                                              |                                              |      |
| :----------------------------------------------------------: | -------------------------------------------- | ---- |
|      `Item.Settings food(FoodComponent foodComponent)`       | 设置物品为食物并设置其食物相关属性           |      |
| `Item.Settings food(FoodComponent foodComponent, ConsumableComponent consumableComponent)` | 设置食品为食物并设置其食物相关属性和特殊效果 |      |

### `FoodComponent`

> 食物组件，封装了食物的常见属性。

**获得食物组件**

> 通过`FoodComponent.Builder`对象的`build`方法构建一个食物组件，在构建前可以设置`Build`对象的一些属性。

```java
new FoodComponent.Builder().build()
```

**设置食物组件属性**

> `FoodComponent.Builder`提供了一些方法，用于设置食物相关属性,如饥饿点数，饱和点数

| 方法                                                         | 描述                             |
| :----------------------------------------------------------- | :------------------------------- |
| `FoodComponent.Builder nutrition(int nutrition)`             | 设置物品将补充的饥饿点数。       |
| `FoodComponent.Builder saturationModifier(float saturationModifier)` | 设置物品将添加的饱和点数量。     |
| `FoodComponent.Builder alwaysEdible()`                       | 无论饥饿程度如何，都可以吃掉物品 |

- 饱和度计算公式为`nutrition * saturationModifier * 2.0F`

**`FoodComponents`**

> 封装了原版所有食物所用到的`FoodCoponent`

### `ConsumableComponent`

> 可消耗组件,定义消耗品(如食物)被消耗或使用后产生的效果。`ConsumableComponents`中封装了原版已有的特殊效果

**获得可消耗组件**

> 提供了`ConsumableComponent.Builder`静态内部类构建可消耗组件

```
ConsumableComponent.builder().build()
```

- `ConsumableComponent`未私有化其构造方法，也可以通过构造方法创建实例，但是构建器更加清晰

**设置可消耗组件属性**

> `ConsumableComponent.Builder`支持链式编程,通过构造器设置属性并调用`build`方法构造对象

|                                                              |                            |      |
| :----------------------------------------------------------: | -------------------------- | ---- |
| `ConsumableComponent.Builder consumeEffect(ConsumeEffect consumeEffect)` | 设置消耗品的特殊效果       |      |
| `ConsumableComponent.Builder consumeSeconds(float consumeSeconds)` | 设置消耗品被使用所需的时间 |      |

```java
public static final ConsumableComponent POISON_FOOD_CONSUMABLE_COMPONENT = ConsumableComponents.food()
		// The duration is in ticks, 20 ticks = 1 second
		.consumeEffect(new ApplyEffectsConsumeEffect(new StatusEffectInstance(StatusEffects.POISON, 6 * 20, 1), 1.0f))
		.build();
public static final FoodComponent POISON_FOOD_COMPONENT = new FoodComponent.Builder()
		.alwaysEdible()
		.build();

new Item.Settings().food(POISON_FOOD_COMPONENT, POISON_FOOD_CONSUMABLE_COMPONENT)
```

- `ApplyEffectsConsumeEffect`是一个行为层对象,持有一个或多个 `StatusEffectInstance`和一个几率参数,表示有概率给予效果
- `StatusEffectInstance`描述一个“具体的状态效果”实例。

## 自定义工具

1. 首先创建工具材料对象，用于设置工具的通用属性

   ```java
   public class ModToolMaterials {
       public static ToolMaterial myToolMaterial = new ToolMaterial(BlockTags.INCORRECT_FOR_WOODEN_TOOL, 59, 2.0F, 0.0F, 15, ItemTags.WOODEN_TOOL_MATERIALS);
   }
   ```

   | 参数                      | 描述                                                         |
   | :------------------------ | :----------------------------------------------------------- |
   | `incorrectBlocksForDrops` | 如果一个方块具有`incorrectBlocksForDrops`标签，这就意味着当你对这个方块使用由这个`ToolMaterial`创建的工具，这个方块不会有任何掉落物。 |
   | `durability`              | 该`ToolMaterial`的耐久度。                                   |
   | `speed`                   | 该`ToolMaterial`的挖掘速度。                                 |
   | `attackDamageBonus`       | 该`ToolMaterial`的额外攻击伤害。                             |
   | `enchantmentValue`        | 该`ToolMaterial`在附魔时得到更好附魔的概率。                 |
   | `repairItems`             | 任何具有这个标签的物品可以被用来在铁砧中修复该`ToolMaterial`。 |

2. 根据该材料注册工具并进行设置

   ```java
   //剑
   public static Item GUIDITE_SWORD = register("guidite_sword",Item::new, new Item.Settings().sword(ToolMaterial.WOOD, 3.0F, -2.4F));
   //铲子
   public static final Item WOODEN_SHOVEL = register("wooden_shovel", set tings -> new ShovelItem(ToolMaterial.WOOD, 1.5F, -3.0F, settings));
   //镐
   public static final Item WOODEN_PICKAXE = register("wooden_pickaxe", new Item.Settings().pickaxe(ToolMaterial.WOOD, 1.0F, -2.8F));
   //斧子
   public static final Item WOODEN_AXE = register("wooden_axe", settings -> new AxeItem(ToolMaterial.WOOD, 6.0F, -3.2F, settings));
   //锄头
   public static final Item WOODEN_HOE = register("wooden_hoe", settings -> new HoeItem(ToolMaterial.WOOD, 0.0F, -3.0F, settings));
   ```

3. 添加物品相关的资源文件，与普通物品不同，工具的物品模型应继承`minecraft:item/handheld`父类模型。

## 自定义物品交互

> 实现物品交互需要了解几个类:

**TypeActionResult**

> 用于 `ItemStacks`,负责告诉游戏在事件发生后是否需要替换物品堆叠(item stack)

```java
ItemStack heldStack = user.getStackInHand(hand);
heldStack.decrement(1);
TypedActionResult.success(heldStack);
```

**ActionResult**

> 一个枚举类,告诉游戏事件的状态，*`TypedActionResult`包装了 这个类*

```
ActionResult.PASS
```

### 可被重写的事件

> 这些方法均继承自Item，重写这些方法可以更改物品的交互逻辑

| 方法            | 信息                                                         |
| :-------------- | :----------------------------------------------------------- |
| `postHit`       | 当玩家攻击实体时被调用                                       |
| `postMine`      | 当玩家挖掘方块时被调用                                       |
| `inventoryTick` | 当物品在物品栏(inventory)中时，每一tick调用一次              |
| `onCraft`       | 当物品被合成时调用                                           |
| `useOnBlock`    | 当玩家手持物品右键方块时调用(确切的说是对着方块按下使用按键) |
| `use`           | 当玩家手持物品按下右键时调用(确切的说是按下使用按键)         |

## 自定义附魔效果

## 数组组件

> 在MC中，每一个物品格子(slot)中都会存储一个`ItemStack`。很多时候，需要在物品堆中存储自定义数据以实现一些效果。在1.20+后，`ItemStack`提供了数据组件(DataComponent)替换之前的NBT数据，用于以结构化的方式存储物品堆中的持久数据。MC中 提供了一些原版数据组件供使用，同时，开发者也可以实现自己的数据组件，存储 `ItemStack` 中的自定义数据。

### 注册组件

**一般将所有自定义的组件类型封装到一个类中，并将这个类放在component包下**

```java
public class ModComponents {
    //注册自定义组件类型
    public static final ComponentType<?> MY_COMPONENT_TYPE = Registry.register(
            Registries.DATA_COMPONENT_TYPE,
            Identifier.of(Mymod.MOD_ID, "my_component"),
            ComponentType.<?>builder().codec(null).build()
    );

    public static void initialize() {
        Mymod.LOGGER.info("Registering {} components", Mymod.MOD_ID);
    }
}
```

- 注册组件类型时，需要传入三个参数`(注册类型,标识符,组件实例)`，组件类型实例通过`ComponentType.Builder`内部类获得
- 泛型为该组件类型存储的类型

### 操作数据组件

> 数据组件属于`ItemStack`的属性,`ItemStack`提供了API操作数据组件

**读取数据组件的值**

```java
	<T> T get(ComponentType<? extends T> type)
	//案例
	int clickCount =(int)stack.get(ModComponents.CLICK_COUNT_COMPONENT);
	//带有默认值的读取,如果该物品不具有该组件，则返回默认值fallback
	<T> T getOrDefault(ComponentType<? extends T> type, T fallback)
    int count = stack.getOrDefault(ModComponents.CLICK_COUNT_COMPONENT,0);
```

**判断组件是否存在**

```java
boolean contains(ComponentType<?> type)
//案例
stack.contains(ModComponents.CLICK_COUNT_COMPONENT)
```

**更新组件值**

```java
	//更新组件值的同时返回原数值
	<T> T set(ComponentType<T> type, @Nullable T value)
        
    int oldValue = stack.set(ModComponents.CLICK_COUNT_COMPONENT, newValue);
```

**移除组件**

```

```



### 基本数据组件

> 基本数据组件中存储单个基本数据类型值，如 `int`、`float`、`boolean` 或 `String`。

```java
public static final ComponentType<Integer> CLICK_COUNT_COMPONENT = Registry.register(
		Registries.DATA_COMPONENT_TYPE,
		Identifier.of(FabricDocsReference.MOD_ID, "click_count"),
		ComponentType.<Integer>builder().codec(Codec.INT).build()
);
```

- 对于基本数据组件，可以直接使用`Codec`接口中定义的对应静态字段；更复杂的数据组件需要自定义`codec`

### 为物品添加数据组件

**游戏内**

> 游戏内的`give`命令可以在给予玩家物品的同时为物品添加数据组件

```
/give @p <namespace>:<item_name>[<namespace>:<component_name>=5]
```

- 在组件前加`!`,如`<namespace>:<component_name>=5`表示删除这个组件

**代码**

> 在注册物品时,可以通过`Item.Settings`对象为物品添加数据组件并为组件添加默认值

```java
<T> Item.Settings component(ComponentType<T> type, T value)
//例子
new Item.Settings().component(ModComponents.CLICK_COUNT_COMPONENT, 0)
```

## 物品组

> 物品组即一组同类物品形成的组合，只有添加到物品组的物品才可以在物品栏中搜索到。MC中通过`ItemGroup`类表示一个物品组

### 相关类

#### `ItemGroup`

> 代表一个物品组对象

#### `ItemGroups`

> MC原版中物品组的注册类，封装了原版已存在的物品组的`RegistryKey`.

### 添加物品到原版物品组

> 通过`Fabric API`修改原版物品组。

```java
      	ItemGroupEvents.modifyEntriesEvent(ItemGroups.BUILDING_BLOCKS)
            .register(fabricItemGroupEntries -> {
            fabricItemGroupEntries.add(TutorialItems.CUSTOM_ITEM);
        });
```

- `content`对象提供了`add`,`addAfter`,`addBefore`等方法可以灵活调整新物品在物品组中的位置
- `modifyEntriesEvent`方法接收`RegistryKey`查找物品组，而原版的原版物品组均注册在`ItemGroups`类中，且将`RegistryKey`封装为静态字段

### 创建自定义物品组

> 通过`FabricAPI`提供的 `FabricItemGroup.builder` 方法获得`ItemGroup.Builder`构建器，然后通过构建器设置物品组属性并构建物品组实例，然后注册物品组实例

```java
public static final RegistryKey<ItemGroup> CUSTOM_ITEM_GROUP_KEY = RegistryKey.of(Registries.ITEM_GROUP.getKey(), Identifier.of(FabricDocsReference.MOD_ID, "item_group"));
public static final ItemGroup CUSTOM_ITEM_GROUP = FabricItemGroup.builder()
		.icon(() -> new ItemStack(ModItems.GUIDITE_SWORD))
		.displayName(Text.translatable("itemGroup.fabric_docs_reference"))
		.build();
Registry.register(Registries.ITEM_GROUP, CUSTOM_ITEM_GROUP_KEY, CUSTOM_ITEM_GROUP);
```

- `FabricItemGroup`实质是对`ItemGroup`构建方法的再封装，也可以使用原版方法`ItemGroup.create(ItemGroup.Row location, int column).build()`,`create`方法中的参数用于设置物品组在物品栏中的位置，但貌似不起作用，无论怎么设置，物品组都会以此从左到右，从上到下排放，不会有空缺，因此`FabricItemGroup`取消了这个方法，并为位置属性提供了默认值。

- `ItemGroup.Builder`,支持链式编程,常用方法如下

  |                                                     |                                                              |      |
  | --------------------------------------------------- | ------------------------------------------------------------ | ---- |
  | ` icon(Supplier<ItemStack> iconSupplier)`           | 接收一个返回`ItemStack`对象的函数式接口实例，根据`ItemStack`对象设置物品组的图标 |      |
  | ` entries(ItemGroup.EntryCollector entryCollector)` | 接收一个函数式接口实例,可通过实例方法中的参数向物品组添加物品 |      |
  | `displayName(Text displayName)`                     | 接收一个`Text`对象作为物品组命名                             |      |
  | `ItemGroup build()`                                 | 根据设置构建一个`ItemGroup`实例                              |      |

# 方块

> 即`Block`,同物品一样也是存储在注册表中

## 相关类

### `Block`

> 表示一种方块的类，封装了方块的常用方法与属性

- 原版所有方块全部注册在`Blocks`类中，并将方块作为静态字段存储在注册类中。

### `BlockItem`

> `Item`的子类，方块本身也是一个可以放在物品栏中的物品,`BlockItem`是作为方块到物品的桥梁。

### `BlockSetType`

> 一个记录类，用于定义“一组方块的交互特性集合”。内部封装了原版材质(如铁，木头)衍生方块族的交互特性集合。

### `PillarBlock`

### `DoorBlock`

- `DoorBlock`需要单独提供物品模型，因为其方块模型均不适合做物品模型。

### **`TrapdoorBlock`**

> `Block`的子类，代表活版门方块

**构造方法**

```java
TrapdoorBlock(BlockSetType type, AbstractBlock.Settings settings)
```

- `BlockSetType`是一个记录类，用于定义“一组方块的交互特性集合”，如声音、能否被红石触发等特性。

### `BlockFamily`

> `BlockFamily`是一个组织数据的类，将方块及其衍生物方块(比如方块对应的栅栏,门,活版门等)组织到一个`BlockFamily`中,在数据生成时可以通过`BlockFamily`快速生成一个家族的相关数据。

- 原版所有方块族全部封装到了`BlockFamilies`中。

## 自定义方块

> 自定义方块与自定义物品一样，只要将方块实例注册，就可以使用此类方块。

### 普通方块

#### **`Block`**

**常用属性**

|                    |                                             |      |
| :----------------: | :-----------------------------------------: | ---- |
| `VoxelShape SHAPE` | 定义物体的碰撞箱，通过`getOutlineShape`返回 |      |
|                    |                                             |      |
|                    |                                             |      |

**常用方法**

|                                 |                                         |      |
| :-----------------------------: | :-------------------------------------: | ---- |
|  `VoxelShape getOutlineShape`   | 定义物体的碰撞箱，默认实现为一个满方块  |      |
| `BlockRenderType getRenderType` | 返回一个类型为`BlockRenderType`的枚举值 |      |
|     `void onStateReplaced`      |                                         |      |
|      `ActionResult onUse`       |           玩家右击方块时调用            |      |
|                                 |                                         |      |

### 自定义作物

> 作物也是一种特殊的方块。

**`CropBlock`**

> 所有作物方块的父类，定义了作物方块的特殊属性与方法。

- 常用属性

  |                                      |                              |      |
  | :----------------------------------: | :--------------------------: | ---- |
  |          `int MAX_AGE = 7`           | 定义作物的生长阶段(从0开始)  |      |
  | `IntProperty AGE = Properties.AGE_7` |      定义作物的生长阶段      |      |
  |     `VoxelShape[] SHAPES_BY_AGE`     | 定义作物在不同阶段的显示体积 |      |
  |     `MapCodec<CropBlock> CODEC`      |                              |      |

- 常用方法

  |      |      |      |
  | ---- | ---- | ---- |
  |      |      |      |
|      |      |      |
  |      |      |      |
  
  

1. 创建作物类继承`CropBlock`，重写其中所需的方法与属性。

   ```java
   public class StrawberryCropBlock extends CropBlock {
       public static final int MAX_AGE = 5;
       public static final IntProperty AGE = Properties.AGE_5;
       public static final MapCodec<CropBlock> CODEC = createCodec(StrawberryCropBlock::new);
       private static final VoxelShape[] SHAPES_BY_AGE = Block.createShapeArray(5, age -> Block.createColumnShape(16.0, 0.0, 2 + age));
   
       public StrawberryCropBlock(Settings settings) {
           super(settings);
       }
   
       @Override
       protected void appendProperties(StateManager.Builder<Block, BlockState> builder) {
           builder.add(AGE);
       }
   
       @Override
       protected IntProperty getAgeProperty() {
           return AGE;
       }
   
       @Override
       public int getMaxAge() {
           return MAX_AGE;
       }

       @Override
       public MapCodec<? extends CropBlock> getCodec() {
           return CODEC;
       }
   
       @Override
       protected VoxelShape getOutlineShape(BlockState state, BlockView world, BlockPos pos, ShapeContext context) {
           return SHAPES_BY_AGE[this.getAge(state)];
       }
   }
   ```
   
   - 如果自定义作物的生长阶段不为8，则需重新定义`MAX_AGE`  ,`AGE `, `SHAPES_BY_AGE` 这三个常量，并重写`getAgeProperty`,`getMaxAge`,`getOutlineShape`方法返回新的常量。
   
2. 注册自定义作物方块和其种子物品

   ```java
       public static final Block STRAWBERRY = register("strawberry",StrawberryCropBlock::new, AbstractBlock.Settings.copy(Blocks.POTATOES));
       public static final Item STRAWBERRY_SEEDS = register("strawberry_seeds",settings -> new BlockItem(ModBlocks.STRAWBERRY,settings),new Item.Settings().useItemPrefixedTranslationKey());
   ```

3. 在客户端添加作物相关特殊渲染

   ```java
   BlockRenderLayerMap.putBlock(ModBlocks.STRAWBERRY,BlockRenderLayer.CUTOUT);
   ```

   

## 方块相关资源文件

- **纹理**:`assets/mod-id/textures/block/<img_name>.png`
- 模型:`assets/<mod-id>/models/block/<block_name>.json`
- **物品模型映射**:`assets/<mod-id>/items/<item_name>.json`
- **命名翻译键**:`assets/<mod-id>/lang/zh_cn.json`
- **方块状态**:`assets/<mod-id>/blockstate/<block_name>.json`,方块状态文件的定义较复杂，详解[方块状态](##方块状态)

### **纹理**

- 方块的纹理均放置在`assets/mod-id/textures/block`中
- 纹理为一个png格式的图片，图片名称必须与方块id一致

### 模型

> 方块模型定义了方块的外观和视觉效果，它指定了方块的纹理，模型模型平移、旋转、缩放和其他属性

- 模型由一个`json`文件定义，位于`assets/<mod-id>/models/block/<block_name>.json`

**简单模型文件**

> 对于一些简单的模型,通过继承已存在的模型(有必要时可以覆盖父类模型的一些属性)，之后指定纹理即可。

```json
//最简单的方块模型，6个面均使用同一个纹理
{
  "parent": "minecraft:block/cube_all",
  "textures": {
    "all": "fabric-docs-reference:block/condensed_dirt"
  }
}
//柱模型，具有水平和垂直两种状态，minecraft:block/cube_column

//柱模型的水平状态,block/cube_column_horizontal
{
  "parent": "minecraft:block/cube_column",
  "textures": {
    "side": "mymod:block/log_side",
    "end": "mymod:block/log_top"
  }
}
```

**`parent`**

- 含义：继承其它模型（比如 `block/cube_all`, `block/cube`等）。

- 可用值：任意已有模型路径（例如 `minecraft:block/cube_all` 或 `block/cube_column` 或 自定义模型路径）。

- 用途：快速复用 vanilla 定义的方块立方体、柱体、十字（cross）等模板。若不想写 `elements`，常用 `parent`。

- 继承父模型的内容

  | 节点               | 是否继承  | 可在子模型覆盖 | 说明                                                    |
  | ------------------ | --------- | -------------- | ------------------------------------------------------- |
  | `elements`         | ✅         | 完全覆盖       | 定义几何体，如果子模型定义了，就替代父模型。            |
  | `textures`         | ✅         | 可覆盖/新增    | 子模型可以覆盖父模型的纹理变量或新增 `particle`。       |
  | `ambientocclusion` | ✅         | 可覆盖         | 决定是否启用环境遮蔽 AO。                               |
  | `display`          | ✅（item） | 可覆盖/新增    | GUI、手持、第一人称、第三人称的显示参数。               |
  | `overrides`        | ✅（item） | 可覆盖/新增    | 用于条件切换模型。                                      |
  | `parent`           | ❌         | 不继承         | 父模型不能继承父的父模型。每个模型只能有一个 `parent`。 |

**`textures`**

- 含义：纹理映射表，内部可自定义多个纹理变量，纹理变量的值为`<namespace>:<type>/<texture_name>`
- 对于简单模型，通过继承父类的纹理变量并覆盖纹理变量的值来快速定义自己的模型。

**复杂模型文件**

```json
{
  "parent": "...",
  "ambientocclusion": "true/false",
  "display": {
    "<position>": {
      "rotation": [0.0, 0.0, 0.0],
      "translation": [0.0, 0.0, 0.0],
      "scale": [0.0, 0.0, 0.0]
    }
  },
  "textures": {
    "particle": "...",
    "<texture_variable>": "..."
  },
  "elements": [
    {
      "from": [0.0, 0.0, 0.0],
      "to": [0.0, 0.0, 0.0],
      "rotation": {
        "origin": [0.0, 0.0, 0.0],
        "axis": "...",
        "angle": "...",
        "rescale": "true/false"
      },
      "shade": "true/false",
      "light_emission": "...",
      "faces": {
        "<key>": {
          "uv": [0, 0, 0, 0],
          "texture": "...",
          "cullface": "...",
          "rotation": "...",
          "tintindex": "..."
        }
      }
    }
  ]
}
```

**`ambientocclusion`(可选)**

```json
{
  "ambientocclusion": "true/false"
}
```

- 含义：是否开启环境遮蔽（AO）。
- 值：`true`（默认）或 `false`。
- 影响：开启时立方体角落/缝隙渲染更暗；平铺或发光纹理常设为 `false`。

**`textures`**

```json
{
  "textures": {
    "particle": "...",
    "<texture_variable>": "..."
  }
}
```

- `particle` 不属于任何父模型定义的变量,是一个特殊键，它控制方块破碎粒子和物品悬浮时的粒子纹理，通常与主纹理一致，但你也可以单独指定。

**`elements`**

`elements` 是最重要的栏目，用于定义任意形状的立方体“元素”（cube）。是一个数组，每个元素定义一个轴对齐立方体及其六个面如何贴图、旋转等。

```json
"elements": [
  {
    "from": [0, 0, 0],
    "to":   [16, 16, 16],
    "rotation": { "origin": [8,8,8], "axis": "y", "angle": 45, "rescale": true },
    "shade": true,
    "faces": {
      "north": { "texture": "#side", "uv": [0,0,16,16], "cullface": "north", "rotation": 0 },
      "up":    { "texture": "#top",  "uv": [0,0,16,16] }
    }
  }
]
```

**`from` / `to`**

- 含义：定义立方体在局部坐标系中的最小点和最大点。
- 值：数组 `[x, y, z]`，范围通常 `0` 到 `16`（1个方块为 16 个像素单位）。
- 示例：`[0,0,0]` 到 `[16,16,16]` 表示覆盖整个格子。更小的值用于凹入、凸起或薄板。

**`rotation`**（可选）

- 含义：对该 element 在模型坐标内做旋转（注意这不是 blockstate 的旋转，而是模型内部的局部旋转）。
- 字段：
  - `origin`: 旋转中心 `[x,y,z]`（通常 8,8,8 对称中心）
  - `axis`: `"x"`、`"y"` 或 `"z"`
  - `angle`: 旋转角度，只能是 `-45`、`-22.5`、`0`、`22.5`、`45`（但大多数用 0/22.5/45，Minecraft 对角旋转受限）。**注意**：某些版本仅允许 22.5 或 45 等特定值。
  - `rescale`: `true/false`（是否对立方体做缩放以匹配旋转后尺寸；默认 `false`）
- 用途：制作斜面、倾斜板等。

**`shade`**

- 含义：是否使用阴影着色
- 值：`true`（默认）或 `false`。对像素风格/平光材质常设为 `false`。

**`faces`**

- 含义：定义立方体六个面的贴图、UV、裁剪、旋转等。键名是 `down`、`up`、`north`、`south`、`west`、`east`。
- 每个面的字段说明（常见）：
  - `texture`：必须字段。可以是 `"#name"`（引用 `textures` 里定义的纹理名）或直接使用纹理标识符（`mymod:block/xxx`）。
  - `uv`：数组 `[u1, v1, u2, v2]`，定义贴图在纹理图上的子区域。坐标以像素为单位（0—16 或更大，如果贴图是 atlas）。顺序通常为左上（u1,v1）到右下（u2,v2）。
  - `cullface`：`"north"| "south"| "east"| "west"| "up"| "down"`。指定当与相邻方块相接触时进行背面剔除（优化）；常用于完整方块的面。
  - `rotation`：`0|90|180|270` —— 对该面的纹理作顺时针旋转（以 90° 为步进）。注意这与 element 的 `rotation` 不同。
  - `tintindex`：整数或 `0`/`-1`，用于调色（biome tinting）。`-1` 或省略表示不染色。`0` 表示使用 `tintindex` 0。
  - `ambientocclusion`：可以在 face 级别覆盖模型级 AO（并不常用）。

### **物品模型映射**

> 物品模型映射指定了方块作为物品时需要渲染的模型(即在物品栏中，在手中，作为掉落物时的渲染模型)

- 只有在注册方块的同时注册了 `Item` 时，才需要创建物品模型映射
- 在1.21.4前，通常会在`models/item`下为方块的物品渲染再创建一个模型文件，直接继承方块模型。

```json
{
  "model": {
    "type": "minecraft:model",
    "model": "fabric-docs-reference:block/condensed_dirt"
  }
}
```

- 一般直接复用方块的模型作为物品的模型

### **命名翻译键**

> MC中通过为方块设置翻译键为方块命名，可以为方块关联多个语言的翻译键以实现国际化。所有方块的翻译键均以`block`开头

- 中文翻译键位于`assets/mod-id/lang/zh_cn.json`
- 英文翻译键位于`assets/mod-id/lang/en_us.json`

```
{
  "block.<mod_id>.<block_id>": "方块"
}
```

### 战利品表

> 为了能在方块被破坏时掉落方块，需要实现方块的战利品表

- 战利品表文件应为 `data/mod-id/loot_table/blocks/<block_name>.json`

  ```json
  {
    "type": "minecraft:block",
    "pools": [
      {
        "rolls": 1,
        "entries": [
          {
            "type": "minecraft:item",
            "name": "fabric-docs-reference:condensed_dirt"
          }
        ],
        "conditions": [
          {
            "condition": "minecraft:survives_explosion"
          }
        ]
      }
    ]
  }
  ```


> 

## 方块状态

> 方块状态是附加到 Minecraft 世界中的单个方块上的一段数据,可用于指示游戏基于当前方块的状态要渲染哪个模型。

**原版存储在方块状态中的属性示例:**

- Rotation：主要用于原木方块和其他自然方块中,旋转属性无法沿z轴旋转，只能沿x，y轴旋转.
- Activated：主要用于红石装置方块和类似于熔炉、烟熏炉的方块中。
- Age：用于农作物、植物、树苗、海带等方块中使用。

**方块状态的出现避免了在方块实体中存储 NBT 数据的需要，既减小了世界大小，也防止产生 TPS 问题。**

### 自定义方块状态

> 方块状态本质是每个方块独有的属性的集合，可以以复用原版的属性,也可自定义新的属性，

#### **添加属性**

1. 自定义方块类，添加属性字段

   ```java
   public class PrismarineLampBlock extends Block {
   	public static final BooleanProperty ACTIVATED = BooleanProperty.of("activated");
   
   }
   ```

2. 重写方块类的`appendProperties`方法，注册属性

   ```java
   @Override
   protected void appendProperties(StateManager.Builder<Block, BlockState> builder) {
       //所有状态均会被保存到BlockState对象中
   	builder.add(ACTIVATED);
   }
   ```

3. 在方块构造方法中设置属性的默认状态

   ```java
   public PrismarineLampBlock(Settings settings) {
   	super(settings);
   
   	// Set the default state of the block to be deactivated.
   	setDefaultState(getDefaultState().with(ACTIVATED, false));
   }
   ```

#### **操作属性**

> 所有属性被封装在`BlockState`对象中,大多数需要操作属性的方法均会自动注入`BlockState`对象

```
//访问属性
state.get(ACTIVATED);
//修改属性
world.setBlockState(pos, state.with(ACTIVATED, !activated));
```

- 每个 `Block` 在注册时都会**预生成并缓存所有可能的 BlockState 组合**，这些状态被存放在一个类似 `Map<PropertyMap, BlockState>` 的结构中。
- `with`方法计算出新的状态并从Map中返回这个状态，再调用`world.setBlockState`将这个方块状态写到世界中
- `BlockState`对象是不可变的

### 方块状态映射

> 方块状态是可以动态变化的，根据方块状态的不同，可以动态渲染不同的模型

- 定义这种状态到模型映射关系的文件称为状态映射文件

- 方块状态映射文件应为`assets/mod-id/blockstates/<block_name>.json`,

**无属性的状态映射文件**

```json
{
  "variants": {
    "": {
      "model": "fabric-docs-reference:block/condensed_dirt"
    }
  }
}
```

- 即使方块并不具有属性，也需要添加状态映射文件才可以成功渲染，此时映射文件如上

**具有属性的状态映射文件**

- 方式一

  ```json
  {
    "variants": {
      "<property>=<value>[,<property2>=<value2>...]": {
        "model": "<namespace>:<path>",
        "x": <rotation_x>,
        "y": <rotation_y>,
        "uvlock": <true|false>,
        "weight": <int>
      }
    }
  }
  ```

  - 多个属性用`,`隔开,但必须列出所有属性组合，否则在某些属性组合时会渲染失败

  | 字段      | 说明                                   |
  | --------- | -------------------------------------- |
  | `model`   | 模型的标识符                           |
  | `x` / `y` | 旋转角度，范围：0、90、180、270        |
  | `uvlock`  | 是否锁定UV（防止旋转导致纹理方向变化） |
  | `weight`  | 权重（用于随机模型，比如草方块）       |

- 方式二

  ```
  {
    "multipart": [
      {
        "when": { "axis": "y" },
        "apply": { "model": "my_mod:block/pillar_vertical" }
      },
      {
        "when": { "axis": "x" },
        "apply": { "model": "my_mod:block/pillar_horizontal", "x": 90, "y": 90 }
      },
      {
        "when": { "axis": "z" },
        "apply": { "model": "my_mod:block/pillar_horizontal", "x": 90 }
      }
    ]
  }
  ```

  - `multipart` 会自动组合未指定的其他属性,从而省去重复定义

## 方块实体

> **方块实体（Block Entity）** 是一种能**保存额外数据和逻辑行为**的升级版方块,是普通方块的扩展

- 方块状态静态存储在区块中，**不会随时间变化**（除非你显式调用 `world.setBlockState`）,而方块实体所存储的数据可以是复杂，动态的数据
- 方块状态可以做到的，都可以通过方块实体做到，而方块实体可以做到的，方块状态未必可以做到
- 方块实体本身也具有方块状态

**`Inventory`**

> 此接口定义了实体操作库存的方法，通过为实体添加`DefaultedList#ofSize(int， Object)`字段给实体添加容纳物品堆的容器，再实现`Inventory`接口操作``DefaultedList`中的物品堆，从而为实体添加库存系统。

- 通常不直接实现`Inventory`，而是实现他的子接口或继承其实现类。如果为实体添加库存功能，则实现`VehicleInventory`接口；如果为方块实体添加库存功能,则继承`LockableContainerBlockEntity`类
- `DefaultList`中的每一个位置被视为一个库存插槽(slot)，MC按照索引(从0开始)为插槽编号。

**`SidedInventory`**

> 是 `Inventory` 的扩展接口，为方块实体添加存储系统的同时，额外定义了来自不同方向的物品插入与提取规则

**`BlockEntityTicker`**

- 一个函数式接口，配合`BlockWithEntity`的`getTicker`方法实现方块每一刻持续调用逻辑的功能
- 通常将接口的方法实现为静态方法

**`BlockWithEntity`**

> `Block`的子类，额外提供了一些与方块所对应方块实体类相互协作的方法。

|                                                              |                                                              |                                                             |
| :----------------------------------------------------------: | ------------------------------------------------------------ | ----------------------------------------------------------- |
| `getTicker(World world, BlockState state, BlockEntityType<T> type)` | 根据`type`返回对应的`BlockEntityTicker`，如果均不符合，则返回null,此时间刻不调用任何逻辑。 | 需要方块对应的方块实体类配合实现`BlockEntityTicker`接口使用 |
|                                                              |                                                              |                                                             |
|                                                              |                                                              |                                                             |

**`BlockEntity`**

> 方块实体，用于定义一个方块实体

### 添加方块实体

1. 创建方块实体类，继承`BlockEntity`

   ```
   public class CounterBlockEntity extends BlockEntity {
       public CounterBlockEntity(BlockPos pos, BlockState state) {
           super(ModBlockEntities.COUNTER_BLOCK_ENTITY, pos, state);
       }
   }
   ```

2. 添加方块实体所对应的方块类，要继承`BlockEntityProvider`或`BlockWithEntity`,并重写`createBlockEntity`方法。这是方块实体对应的方块

   ```
   public class CounterBLock extends BlockWithEntity {
       public CounterBLock(Settings settings) {
           super(settings);
       }
   
       @Override
       protected MapCodec<? extends BlockWithEntity> getCodec() {
           return createCodec(CounterBLock::new);
       }
   
       @Override
       public @Nullable BlockEntity createBlockEntity(BlockPos pos, BlockState state) {
           return new CounterBlockEntity(pos, state);
       }
   }
   
   ```

   - `BlockWithEntity`相比于`BlockEntityProvider`提供了一些新方法
   - 此方块类需要注册，注册方法同普通方块

3. 注册方块实体，注册类型为`BlockEntityType`。原版方块实体注册在`BlockEntityType`类中。

   ```java
   public static final BlockEntityType<CounterBlockEntity> COUNTER_BLOCK_ENTITY =
   		register("counter", CounterBlockEntity::new, ModBlocks.COUNTER_BLOCK);
   
   private static <T extends BlockEntity> BlockEntityType<T> register(
   		String name,
   		FabricBlockEntityTypeBuilder.Factory<? extends T> entityFactory,
   		Block... blocks
   ) {
   	Identifier id = Identifier.of(FabricDocsReference.MOD_ID, name);
   	return Registry.register(Registries.BLOCK_ENTITY_TYPE, id, FabricBlockEntityTypeBuilder.<T>create(entityFactory, blocks).build());
   }
   ```
   
   - `BlockEntityType`私有了其创建实例的方法，可以通过`Fabric API`中的`FabricBlockEntityTypeBuilder`创建`BlockEntityType`。

### 操作方块实体

1. 可以直接向方块实体类中添加字段以及操作字段的方块

   ```
     private int clicks = 0;
     public int getClicks() {
     	return clicks;
     }
     
     public void incrementClicks() {
     	clicks++;
     	markDirty();
     }
   ```

     - `markDirty()`告诉游戏该实体的数据已更新

2. 在方块实体对应的方块类中编写方法调用方块实体类中的方法

   ```java
   @Override
   protected ActionResult onUse(BlockState state, World world, BlockPos pos, PlayerEntity player, BlockHitResult hit) {
   	if (!(world.getBlockEntity(pos) instanceof CounterBlockEntity counterBlockEntity)) {
   		return super.onUse(state, world, pos, player, hit);
   	}
   
   	counterBlockEntity.incrementClicks();
   	player.sendMessage(Text.literal("You've clicked the block for the " + counterBlockEntity.getClicks() + "th time."), true);
   
   	return ActionResult.SUCCESS;
   }
   ```

**保存和加载数据**

> 方块实体的属性在重启游戏后会清空，通过在游戏保存时将其序列化为 NBT，并在加载时反序列化来实现保存其数据

```java
public class CounterBlockEntity extends BlockEntity {
    public CounterBlockEntity(BlockPos pos, BlockState state) {
        super(ModBlockEntities.COUNTER_BLOCK_ENTITY, pos, state);
    }
    private int clicks = 0;
    public int getClicks() {
        return clicks;
    }

    public void incrementClicks() {
        clicks++;
        markDirty();
    }

    @Override
    protected void writeData(WriteView view) {
        view.putInt("clicks", clicks);
        super.writeData(view);
    }

    @Override
    protected void readData(ReadView view) {
        super.readData(view);
        clicks = view.getInt("clicks", 0);
    }
}
```

## 方块设置

> 在 Minecraft 中，**每个方块都有一份静态的配置**，这些配置决定了方块的：

- 物理特性（硬度、爆炸抗性、透明度）
- 交互特性（可否被活塞推动、是否导电）
- 声音、材质、光照等表现

**这些配置是通过在`Block`构造方法中传入 `AbstractBlock.Settings`对象构建的**

```
new Block(
    AbstractBlock.Settings.create()
        .strength(1.5f, 6.0f)
        .luminance(10)
        .sounds(BlockSoundGroup.STONE)
)
```

- 设置对象支持链式编程

### 获得设置对象

> 获得设置对象的方法有两种。

```java
//创建一个空的Settings对象
AbstractBlock.Settings.create()
//从已有方块中复制一份
Abstract.Settings copy(AbstractBlock block)
```

### 常用设置

> 通过调用`Abstract.Settings`对象的各种方法对物品属性进行设置

**方块照明**

```java
AbstractBlock.Settings.create()
            .luminance(new ToIntFunction<BlockState>() {
                @Override
                public int applyAsInt(BlockState value) {
                    return 0;
                }
            })
```

- 提供了动态控制方块照明度的方法,当`BlockState`状态改变时，可以选择更改照明度
- 0为不亮，数字越大照明值越高。

**硬度(挖掘速度)与爆炸抗性**

```java
//分别设置硬度与爆炸抗性
Settings strength(float hardness, float resistance)
//设置硬度与爆炸抗性，两者值相同
Settings strength(float strength)
//设置硬度
Settings hardness(float hardness)
//设置爆炸抗性
Settings resistance(float resistance)
```

**推荐挖掘工具**

```java
AbstractBlock.Settings.create()
            .requiresTool());
```

- 让方块只能被特定类型的工具挖掘或者特定类型的工具挖掘更快

- 需要在 `data/minecraft/tags/block/mineable/` 文件夹内添加不同工具的标签文件，其中文件的名称取决于使用的工具的类型

  - `hoe.json`（锄）

  - `axe.json`（斧）

  - `pickaxe.json`（镐）

  - `shovel.json`（锹）

    ```
    {
      "replace": false,
      "values": ["fabric-docs-reference:condensed_dirt"]
    }
    ```

- 如果在添加标签文件的同时设置`requiresTool`，则只能使用合适工具挖掘，否则效果为合适工具挖掘更快

**挖掘等级**

- 无需在`AbstractBlock.Settings`中配置

- 在`data/minecraft/tags/block/` 文件夹中创建挖掘等级标签文件，并遵循以下格式：
  - `needs_stone_tool.json` - 最低需要石质工具
  - `needs_iron_tool.json` - 最低需要铁质工具
  - `needs_diamond_tool.json` - 最低需要钻石工具
- 在标签文件中注册方块即可

```json
{
  "replace": false,
  "values": ["fabric-docs-reference:condensed_dirt"]
}
```

**非实心**

```
AbstractBlock.Settings.create().nonOpaque();
```

- 控制方块是否会 **遮挡光线和实体渲染**，常用于门或活版门之类的具有透明的方块

- 需要配合渲染API使用,在客户端初始化方法中加入代码,否则透明处渲染为黑色

  ```java
  BlockRenderLayerMap.putBlock(ModBlocks.ICE_ETHER_TRAPDOOR,BlockRenderLayer.CUTOUT);
  BlockRenderLayerMap.putBlock(ModBlocks.ICE_ETHER_DOOR,BlockRenderLayer.CUTOUT);
  ```


**无碰撞箱**

```java
AbstractBlock.Settings.create()
			.noCollision()
```

**立即破坏**

```
AbstractBlock.Settings.create()
		.breakInstantly()
```

- 本质是将方块的`stregnth`设置为0

**活塞推动时的行为**

```java
AbstractBlock.Settings.create()
		.pistonBehavior(PistonBehavior.DESTROY)
```

- `PistonBehavior`为一个枚举类

**开启随机刻机制**

```

```

- 游戏每刻会从已加载的区块中挑选部分随机选出若干个方块，对于那些开启随机刻机制的方块，调用其`randomTick`方法

**设置声音组**

```
AbstractBlock.Settings.create()
		.sounds(BlockSoundGroup)
```



# 实体

> 实体（Entity）是游戏世界里的一种可以根据附加的逻辑移动的物体,包括矿车，箭，船

## 相关类

### `LivingEntity`

> 生物实体,表示拥有生命值，并且可以造成伤害的实体,有多个子类。

#### `HostileEntity` 

> 敌对实体，用于僵尸、苦力怕、骷髅等

#### `AnimalEntity` 

> 动物实体，用于羊、牛、猪等

#### `WaterCreatureEntity` 

> 水生物实体，用于可以游泳的实体

#### `FishEntity` 

> 鱼实体，用于鱼

#### `PlayerEntity`

> 表示一个玩家实体

**成员方法**

|                                                  |                                      |      |
| :----------------------------------------------: | :----------------------------------: | ---- |
|         `PlayerInventory getInventory()`         | 获得玩家的物品栏(包括装备栏，副手槽) |      |
| `ItemStack getEquippedStack(EquipmentSlot slot)` | 获得`slot`(枚举值)指定装备栏的物品堆 |      |
|                                                  |                                      |      |
|                                                  |                                      |      |



## 添加实体

> 在1.21.2版本后

**1️⃣ 创建实体类**

- 新实体通常继承自以下类之一：
  - `LivingEntity`：拥有生命值、伤害等基础功能。
  - `MobEntity`：包含 AI、移动控制。
  - `PathAwareEntity`：包含寻路系统，适合大多数 AI 生物。

```
public class CubeEntity extends PathAwareEntity {
    public CubeEntity(EntityType<? extends PathAwareEntity> entityType, World world) {
        super(entityType, world);
    }

    // 定义实体默认属性
    public static DefaultAttributeContainer.Builder createCubeAttributes() {
        return PathAwareEntity.createMobAttributes()
            .add(EntityAttributes.MAX_HEALTH, 20.0)
            .add(EntityAttributes.MOVEMENT_SPEED, 0.25)
            .add(EntityAttributes.ATTACK_DAMAGE, 2.0);
    }
}
```

**要点：**

- `DefaultAttributeContainer.Builder` 用于注册生命值、速度、攻击力等。
- 如果不注册属性，游戏会在生成实体时崩溃。

------

**2️⃣ 在服务端注册实体类型**

- 实体类型通过 Fabric API 注册到 `Registries.ENTITY_TYPE`。
- 设置生成类别（`SpawnGroup`）和碰撞箱大小（`EntityDimensions`）。

```java
public static final EntityType<CubeEntity> CUBE = Registry.register(
    Registries.ENTITY_TYPE,
    Identifier.of(MOD_ID, "cube"),
    EntityType.Builder.create(SpawnGroup.CREATURE, CubeEntity::new)
        .dimensions(EntityDimensions.fixed(0.75f, 0.75f))
        .build()
);

// 注册默认属性
FabricDefaultAttributeRegistry.register(CUBE, CubeEntity.createCubeAttributes());
```

**要点：**

- `SpawnGroup` 决定实体的生成类别（动物、怪物等）。
- `dimensions` 决定碰撞体积大小。

------

**3️⃣ 创建客户端渲染状态 `EntityRenderState`**

- 新版渲染体系中，引入了 `EntityRenderState`，独立于实体逻辑。
- 所有动画或渲染数据都放在这里。

```
public class CubeEntityRenderState extends LivingEntityRenderState {
    public float animationProgress;
}
```

**要点：**

- 不要把逻辑放在渲染状态中，只存储渲染相关信息。
- 通过 `updateRenderState()` 方法从实体同步数据。

------

**4️⃣ 创建实体模型类 `EntityModel<EntityRenderState>`**

- 模型泛型使用渲染状态而不是实体本身。
- `setAngles()` 使用 `EntityRenderState` 更新动画。

```
public class CubeEntityModel extends EntityModel<CubeEntityRenderState> {
    private final ModelPart base;

    public CubeEntityModel(ModelPart root) {
        super(RenderLayer::getEntityCutout);
        this.base = root.getChild("base");
    }

    public static TexturedModelData getTexturedModelData() {
        ModelData modelData = new ModelData();
        ModelPartData root = modelData.getRoot();

        root.addChild("base",
            ModelPartBuilder.create()
                .uv(0, 0)
                .cuboid(-6F, 12F, -6F, 12F, 12F, 12F),
            ModelTransform.origin(0F, 0F, 0F));

        return TexturedModelData.of(modelData, 64, 64);
    }

    @Override
    public void setAngles(CubeEntityRenderState state) {
        base.yaw = state.animationProgress * 0.1F;
    }
}
```

------

**5️⃣ 创建客户端渲染器 `MobEntityRenderer<Entity, RenderState, Model>`**

```java
public class CubeEntityRenderer extends MobEntityRenderer<CubeEntity, CubeEntityRenderState, CubeEntityModel> {

    private static final Identifier TEXTURE = Identifier.of(MOD_ID, "textures/entity/cube.png");

    public CubeEntityRenderer(EntityRendererFactory.Context context) {
        super(context, new CubeEntityModel(context.getPart(My_modClient.CUBE_LAYER)), 0.5f);
    }

    @Override
    public CubeEntityRenderState createRenderState() {
        return new CubeEntityRenderState();
    }

    @Override
    public void updateRenderState(CubeEntity entity, CubeEntityRenderState state, float tickDelta) {
        super.updateRenderState(entity, state, tickDelta);
        state.animationProgress = entity.age + tickDelta;
    }

    @Override
    public Identifier getTexture(CubeEntityRenderState state) {
        return TEXTURE;
    }
}
```

**要点：**

- 构造器从 `EntityRendererFactory.Context` 获取模型。
- `updateRenderState` 用实体数据更新渲染状态。
- `getTexture` 返回纹理路径（放在 `resources/assets/<modid>/textures/`）。

------

**6️⃣ 注册客户端渲染器和模型层**

```java
public class My_modClient implements ClientModInitializer {

    public static final EntityModelLayer CUBE_LAYER =
        new EntityModelLayer(Identifier.of(MOD_ID, "cube"), "main");

    @Override
    public void onInitializeClient() {
        EntityModelLayerRegistry.registerModelLayer(CUBE_LAYER, CubeEntityModel::getTexturedModelData);
        EntityRendererRegistry.register(My_mod.CUBE, CubeEntityRenderer::new);
    }
}
```

**要点：**

- 客户端专用初始化类，实现 `ClientModInitializer`。
- 所有渲染器注册都必须在这里完成。

------

**7️⃣ 放置资源文件**

```
resources/assets/<modid>/
 ├─ textures/entity/cube/cube.png
 └─ models/entity/cube.json  (可选 BlockBench 导出)
```

- **纹理文件仅在客户端加载**
- 不放在服务端代码或公共模块中

------

**8️⃣ 总结流程图**

1. **定义实体类** → 包含属性、AI、碰撞箱
2. **服务端注册实体类型** → `Registry.register()` + 默认属性
3. **创建渲染状态类** → 保存动画、颜色等渲染信息
4. **创建模型类** → 泛型为 `EntityRenderState`
5. **创建渲染器类** → 继承 `MobEntityRenderer<Entity, State, Model>`
6. **客户端注册渲染器与模型层** → `ClientModInitializer`
7. **放置资源文件** → 纹理和模型在 `resources/assets`

## 自定义交易

>  原版交易封装在`TradeOffers`类中,可以通过`Minin`修改，`Fabric API`也提供了`TradeOfferHelper`对交易进行修改

```java
        public static void registerModCustomTrades() {
            TradeOfferHelper.registerVillagerOffers(VillagerProfession.FARMER,1,(factories) -> {
                factories.add(new TradeOffers.BuyItemFactory(ModItems.STRAWBERRY,5,10,10));
                factories.add(new TradeOffers.SellItemFactory(ModItems.STRAWBERRY_SEEDS,5,10,10));
            });
        }
```

### `TradeOffers`

> 用于定义村民交易的类

**`TradeOffers.Factory`**

> `Factory`对象用于定义交易项，`TradeOffers`中提供了多种内部类定义不同类型的交易项

|                            实现类                            |                                                  |      |
| :----------------------------------------------------------: | ------------------------------------------------ | ---- |
| `BuyItemFactory(ItemConvertible item, int count, int maxUses, int experience, int price) ` | 玩家向村民出售物品的交易项                       |      |
| `SellDyedArmorFactory(Item item, int price, int maxUses, int experience)` | 村民向玩家出售物品的交易项                       |      |
|                     `ProcessItemFactory`                     | 玩家向村民提供物品与绿宝石，村米加工物品的交易项 |      |

## 自定义村民

> 即自定义村民的职业。在原版中，所有村民职业均在`VillagerProfession`中注册。同时`VillagerProfesson`对象也表示一种村民职业。

1. 村民职业同样需要注册，封装村民的注册方法

   ```java
   public class ModVillagerProfessions {
   
       private static RegistryKey<VillagerProfession> of(String id) {
           return RegistryKey.of(RegistryKeys.VILLAGER_PROFESSION, Identifier.of(My_mod.MOD_ID,id));
       }
   
       private static VillagerProfession register(String path, RegistryKey<PointOfInterestType> heldWorkstation, @Nullable SoundEvent workSound) {
           RegistryKey<VillagerProfession> key = of(path);
           return Registry.register(
                   Registries.VILLAGER_PROFESSION,
                   key,
                   new VillagerProfession(
                           Text.translatable("entity." + key.getValue().getNamespace() + ".villager." + key.getValue().getPath()),
                           entry -> entry.matchesKey(heldWorkstation),
                           entry -> entry.matchesKey(heldWorkstation),
                           ImmutableSet.of(), ImmutableSet.of(),workSound)
           );
       }
   }
   ```

2. 村民职业需要绑定一个激活职业的工作站(或者说兴趣点)，这同样需要注册,注册类型为`PointOfInterestType`。`Fabric API`提供了`PointOfInterestHelper`进行注册,否则需要使用`minin`

   ```java
   public class ModPointOfInterestTypes {
       public static final RegistryKey<PointOfInterestType> ICE_ETHER_MASTER = of("ice_ether_master");
    public static RegistryKey<PointOfInterestType> of(String id) {
           return RegistryKey.of(RegistryKeys.POINT_OF_INTEREST_TYPE, Identifier.of(My_mod.MOD_ID,id));
       }
       public static PointOfInterestType register(String id,int ticketCount,int searchDistance, Block... block) {
           return PointOfInterestHelper.register(Identifier.of(My_mod.MOD_ID, id), ticketCount, searchDistance, block);
       }
       public static void initialize() {
           register("ice_ether_master",1,1, ModBlocks.ICE_ETHER_BLOCK);
       }
   }
   ```
   

- 村民职业的贴图文件位于`assets/<namespace>/textures/entity/villager/profession`文件夹下，为`png`图片，名字同注册id

## 状态效果

> 又称效果，是一种可以影响实体的状况， 可以是正面、负面或中性的。游戏本体通过许多不同的方式应用这些效果，如食物和药水等等

- 可以通过`/effect`给实体应用效果

### 自定义状态效果

1. 创建新类继承所有状态效果的基类 `StatusEffect` ,并重写相关方法

   ```java
   public class TaterEffect extends StatusEffect {
   	protected TaterEffect() {
   		// category: StatusEffectCategory - describes if the effect is helpful (BENEFICIAL), harmful (HARMFUL) or useless (NEUTRAL)
   		// color: int - Color is the color assigned to the effect (in RGB)
   		super(StatusEffectCategory.BENEFICIAL, 0xe9b8b3);
   	}
   
   	// 判断每tick是否触发效果
   	@Override
   	public boolean canApplyUpdateEffect(int duration, int amplifier) {
   		// In our case, we just make it return true so that it applies the effect every tick
   		return true;
   	}
   
   	// 效果的真正执行逻辑
   	@Override
   	public boolean applyUpdateEffect(ServerWorld world, LivingEntity entity, int amplifier) {
   		if (entity instanceof PlayerEntity) {
   			((PlayerEntity) entity).addExperience(1 << amplifier); // Higher amplifier gives you experience faster
   		}
   
   		return super.applyUpdateEffect(world, entity, amplifier);
   	}
   }
   ```

   - `canApplyUpdateEffect`在每一刻判断是否添加效果
   - `applyUpdateEffect`

2. 注册自定义的状态效果

   ```java
   public static final RegistryEntry<StatusEffect> TATER =
   			Registry.registerReference(Registries.STATUS_EFFECT, Identifier.of(FabricDocsReference.MOD_ID, "tater"), new TaterEffect());
   ```

### 纹理

- 即状态效果的图标，是 18x18 的 PNG，出现在玩家的物品栏屏幕中。
- 位于`resources/assets/<namespace>/textures/mob_effect/<effect_name>.png`

### 翻译键

- 翻译键位于

```
{
  "effect.<namespace>.<effect_name": "Tater"
}
```

### 应用状态效果

**游戏内**

```
effect give @p fabric-docs-reference:tater
effect clear @p fabric-docs-reference:tater
```

**代码**

- 通过`LivingEntity`对象的`addStatusEffect`为实体添加效果

```java
var instance = new StatusEffectInstance(FabricDocsReferenceEffects.TATER, 5 * 20, 0, false, true, true);
entity.addStatusEffect(instance);
```

|    参数     |             类型              |                             描述                             |
| :---------: | :---------------------------: | :----------------------------------------------------------: |
|  `effect`   | `RegistryEntry<StatusEffect>` |                     代表效果的注册表项。                     |
| `duration`  |             `int`             |             效果的时长，单位为**刻**，而非**秒**             |
| `amplifier` |             `int`             | 效果等级的倍率。 不是与效果的**等级**直接对应，而是有增加的。 比如，`amplifier` 为 `4` => 等级为 `5` |
|  `ambient`  |           `boolean`           | 这个有些棘手， 基本上是指定效果是由环境（比如**信标**）施加的，没有直接原因。 如果是 `true`，HUD内的效果图标会以青色覆盖层的形式出现。 |
| `particles` |           `boolean`           |                        是否显示粒子。                        |
|   `icon`    |           `boolean`           | 是否在 HUD 中显示效果的图标。 效果会在物品栏中显示，无论其设置的属性。 |

# Identifier

> `Identifier`类表示一个用于标识各种内容的标识符,也被称为资源位置，“命名空间 ID”,“位置”，或简称为”ID“

- 标识符的格式为`<namespace>:<path>`
- 命名空间与路径只能包含 ASCII 小写字母 [a-z]、ASCII 数字 [0-9]，以及字符`_`、` .`和 ` -`。路径还可以包含标准路径分隔符` /`。不能使用大写字母
- 命名空间与路径仅用于标识来源，而不是标识用途。不同的对象可以共享相同的命名空间与路径。比如`minecraft:dirt`可以同时被方块和物品使用，方块与其对应的物品使用同名标识符也是模组开发的惯例。

**namespace**

> 命名空间用来标识对象的来源。如两个模组添加了某个同名物品，此时可以通过命名空间区分两者

- 通常约定使用模组ID作为命名空间

- 如果省略命名空间与冒号,则使用默认命名空间`minecraft`

**path**

> 路径用于标识命名空间中的具体对象，有时也会用于引用文件路径，如纹理文件

# 声音

## 相关类

### `SoundEvent`

> 声音事件，是一个音频的载体,封装了一个标识符,通过这个标识符定义到声音文件

### `SoundEvents`

> 注册类，封装并注册原版MC中的所有声音事件，可以直接调用内部封装的声音，或者使用内部的API注册自己的声音

## 调用声音

- 所有声音调用，最终底层都是调用`World`对象的`playSound`方法。
- 如果其他类中封装的声音调用方法不够灵活，可以使用`World`对象中的方法

**Entity**

> 实体对象，封装了多个调用声音的方法

|                                                              |      |                                                              |
| ------------------------------------------------------------ | ---- | ------------------------------------------------------------ |
| `void playSound(SoundEvent sound, float volume, float pitch)` |      | `volue`为音量，取值为`0.0f~1.0f`,如果超出这个值,则使用`1.0f`,仅在位于声源`volume * 16`范围内的方块可以听到声音<br/>`pitch`为音高，取值`(0.5f~1.0f)`,不但改变音高，还会改变声音持续时间 |
|                                                              |      |                                                              |
|                                                              |      |                                                              |

## 自定义声音

1. 提供音频文件，MC只支持`OGG Vorbis `格式的音频，后缀为`.ogg`

   - 音频必须只有单声道

   - 音频文件均存放在`./resources/assets/<namespace>/sounds/`目录下

   - 还需要添加`resources/assets/<namespace/sounds.json`,为音频配置字幕

     ```json
     {
       "metal_whistle": {
         "subtitle": "sound.example-mod.metal_whistle",
         "sounds": [
           "example-mod:metal_whistle"
         ]
       },
       "engine": {
         "subtitle": "sound.example-mod.engine",
         "sounds": [
           "example-mod:engine"
         ]
       }
     }
     ```

     - `subtitle`为字幕，为玩家提供更多关于该声音的信息,通常使用翻译键指定字幕内容
     - 父字段为声音在注册时的名字，`sounds`字段为声音资源文件位置的标识符。

2. 注册声音事件

   ```java
   public class ModSoundEvents {
       public static final SoundEvent EAT = register("eat");
       public static SoundEvent register(String path) {
           Identifier id = Identifier.of(My_mod.MOD_ID, path);
           return Registry.register(Registries.SOUND_EVENT, id,SoundEvent.of(id));
       }
       public static void initialize() {
   
       }
   }
   ```

## 声音组

> 通过`BlockSoundGroup`定义一个声音组,对同一方块相关的不同声音进行整合

```java
BlockSoundGroup(
		float volume, float pitch, SoundEvent breakSound, SoundEvent stepSound, SoundEvent placeSound, SoundEvent hitSound, SoundEvent fallSound
	)
public static final BlockSoundGroup WOOD = new BlockSoundGroup(
		1.0F, 1.0F, SoundEvents.BLOCK_WOOD_BREAK, SoundEvents.BLOCK_WOOD_STEP, SoundEvents.BLOCK_WOOD_PLACE, SoundEvents.BLOCK_WOOD_HIT, SoundEvents.BLOCK_WOOD_FALL
	);
```

- 定义了方块的破坏声音，实体经过方块的声音，放置声音，击打声音，实体掉落到方块的声音

# 战利品表

> 战利品表定义了不同行为会掉落什么物品以及掉落的概率与条件

- 战利品表配置在资源文件夹下的`./data/<namespace>/loot_table>`下，根据不同行为再将战利品表分类到多个文件夹中

  | 类型       | 触发来源       | 作用示例                   |
  | ---------- | -------------- | -------------------------- |
  | `blocks`   | 方块被破坏时   | 石头掉落圆石、树叶掉落树苗 |
  | `entities` | 实体被击杀时   | 僵尸掉落腐肉               |
  | `chests`   | 世界结构生成时 | 地牢箱子、村庄箱子等内容   |
  | `gameplay` | 特殊游戏事件   | 钓鱼、交易、战利品等       |

- 战利品表的配置文件为`json`格式，名字与配置对象一致。

## 配置

### 结构

**战利品表由三个字段构成**

- `type`：由一个标识符指定战利品表的类型
- `pools`：随机池，一个战利品表可以配置多个随机池，每个随机池都会独立计算奖励，加和作为最终掉落物
- `random_sequence`：

```json
{
  "type": "minecraft:block",
  "pools": [...],
  "random_sequence": "minecraft:blocks/acacia_button"
}
```

**随机池(`pools`)是战利品表配置的核心,一个随机池由以下字段组成**

- `entries`:抽取项，是物品的组、序列、可能性，或者仅仅是物品本身。可以配置多个
- `conditions`:条件，满足条件随机池才会生效
- `functions`:函数，定义一些特殊的效果
- `rolls`:抽取次数，为一个浮点数
- `bonus_rolls`:额外抽取次数。

### 方块掉落

- `json`文件要与被配置的方块同名

```json
{
  "type": "minecraft:block",
  "pools": [
    {
      "bonus_rolls": 0.0,
      "conditions": [
        {
          "condition": "minecraft:survives_explosion"
        }
      ],
      "entries": [
        {
          "type": "minecraft:item",
          "name": "minecraft:acacia_planks"
        }
      ],
      "rolls": 1.0
    }
  ],
  "random_sequence": "minecraft:blocks/acacia_planks"
}
```

#### type

- string,定义战利品表的类型，所有方块掉落的类型均是`minecraft:block`

#### pools

- array,奖励池列表，每一个元素都是一个奖励对象

| 奖励对象键名  | 类型     | 是否必填 | 说明                               |
| ------------- | -------- | -------- | ---------------------------------- |
| `rolls`       | `number` | ✅        | 掉落次数（执行几次该池的抽取）     |
| `bonus_rolls` | `number` | ❌        | 附加掉落次数（与“抢夺”附魔等挂钩） |
| `entries`     | `array`  | ✅        | 掉落条目定义（掉落什么）           |
| `conditions`  | `array`  | ❌        | 掉落条件（满足才执行池逻辑）       |
| `functions`   | `array`  | ❌        | 对掉落结果的修改函数               |

**`rolls`**

- 表示这个奖池会被尝试多少次
- 值可以是浮点数，也可以是随机范围,如`{"min":1,"max":3}`
- 每一次尝试都会从当前奖励池的`entries`中选出一个结果

**`bonus_rools`**

- 额外的 roll 次数，通常受附魔或其他属性影响
- 如果不需要，就设为 `0.0`。

**`entries`**

- array,定义奖励内容

|   字段   |                  |
| :------: | :--------------: |
| `"type"` |  定义掉落的种类  |
| `"name"` | 掉落物品的标识符 |

**`entries.type`**

|                       可取值                       | 含义                         |
| :------------------------------------------------: | ---------------------------- |
|                 `"minecraft:item"`                 | 掉落物品（最常见）           |
|              `"minecraft:loot_table"`              | 引用另一个掉落表             |
|                `"minecraft:empty"`                 | 表示什么也不掉               |
| `"minecraft:alternatives"`/ `"minecraft:sequence"` | 高级结构，用于条件分支或组合 |

**`entries.name`**

- 根据`type`键可选值不同，一般是代表物品或其他掉落表的标识符

**`conditions`**

- array。掉落条件，决定该奖池或条目是否生效

|     字段      |  含义  |
| :-----------: | :----: |
| `"condition"` | 条件ID |

**`conditions.condition`**

|            条件 ID             |          说明          |
| :----------------------------: | :--------------------: |
| `minecraft:survives_explosion` | 方块在爆炸时幸存才掉落 |
|  `minecraft:killed_by_player`  |     必须由玩家击杀     |
| `minecraft:entity_properties`  |   实体需满足特定属性   |
|   `minecraft:random_chance`    |      随机概率触发      |

**`functions`**

- array,掉落函数,对掉落物进行修改

|     字段     |            |
| :----------: | :--------: |
| `"function"` | 掉落函数Id |

**`functions.function`**

|           函数 ID           |         作用          |
| :-------------------------: | :-------------------: |
|    `minecraft:set_count`    |     设置掉落数量      |
|   `minecraft:apply_bonus`   | 根据附魔调整掉落数量  |
| `minecraft:explosion_decay` |  按爆炸强度衰减掉落   |
|     `minecraft:set_nbt`     | 为掉落物设置 NBT 数据 |

#### random_sequence

- 用于确定随机序列的种子来源，确保在同一个世界坐标下的方块，掉落结果**可重现且稳定**

## 修改原版战利品表

> `Fabric API`提供`LootTableEvents`类对战利品表进行修改，替换等操作

- `LootTableEvents`依据对战利品表的不同操作封装了`MODIFY`,`REPLACE`等`Event<T>`对象，通过这些对象注册修改行为即可修改原版已经存在的列表。

```java
public class ModLootTableModifier {
    public static final Identifier JUNGLE_TEMPLATE = Identifier.ofVanilla("chests/jungle_template");

    public static void modifyLootTable() {
        LootTableEvents.MODIFY.register(new LootTableEvents.Modify() {
            @Override
            public void modifyLootTable(RegistryKey<LootTable> key, LootTable.Builder tableBuilder, LootTableSource source, RegistryWrapper.WrapperLookup registries) {
                if (JUNGLE_TEMPLATE.equals(key.getValue())) {
                    LootPool.Builder builder = LootPool.builder()
                            .rolls(ConstantLootNumberProvider.create(1.0f))
                            .conditionally(RandomChanceLootCondition.builder(1.0f))
                            .with(ItemEntry.builder(ModItems.DIAMOND_PROSPECTOR))
                            .apply(SetCountLootFunction.builder(UniformLootNumberProvider.create(1.0f, 1.0f)));
                    tableBuilder.pool(builder.build());
                }
            }
        });
    }
}
```



# 标签

> 将一组相同类型的元素（如方块、物品、生物等）归为同一类，以便在游戏逻辑、命令、配方、生成规则等处统一引用

- 标签配置在`data/<namespace>/tags/<tag_type>/path`文件夹下

- 标签配置文件为一个json文件,基本结构如下

  ```json
  {
    "replace": false,
    "values": [
      "minecraft:oak_planks",
      "minecraft:birch_planks",
      "#minecraft:logs"
    ]
  }
  
  ```

  - `replace`字段表示是否覆盖原版文件，默认为`false`
  - `values`字段为一个数组，值为元素标识符或者其他标签的标识符,标签标识符前要加`#`标识。
  - 标签中不但可以添加其他元素，也可以添加其他标签，用于直接引用其他标签中的元素。
  - 在其他文件中引用标签时，标识符为`<namespace>:<path>`,不需要指定标签类型，因为系统可根据上下文推断出标签类型

## 原版标签

### 方块标签

> 所有方块标签均位于`tags/block`目录下

|            标签路径             |             说明             |
| :-----------------------------: | :--------------------------: |
|  `#minecraft:mineable/pickaxe`  |       能用镐子挖的方块       |
|    `#minecraft:mineable/axe`    |       能用斧头挖的方块       |
|  `#minecraft:mineable/shovel`   |       能用铲子挖的方块       |
|    `#minecraft:mineable/hoe`    |       能用锄头挖的方块       |
|  `#minecraft:needs_stone_tool`  |      需要石镐及以上工具      |
|  `#minecraft:needs_iron_tool`   |      需要铁镐及以上工具      |
| `#minecraft:needs_diamond_tool` |     需要钻石镐及以上工具     |
|        `#minecraft:logs`        |   所有木头（原木、去皮木）   |
|       `#minecraft:planks`       |           各类木板           |
|       `#minecraft:leaves`       |           树叶方块           |
|      `#minecraft:saplings`      |             树苗             |
|    `#minecraft:stone_bricks`    |            石砖类            |
|       `#minecraft:stairs`       |         所有楼梯方块         |
|       `#minecraft:slabs`        |         所有台阶方块         |
|       `#minecraft:walls`        |         所有墙类方块         |
|      `#minecraft:buttons`       |     所有按钮（木、石等）     |
|       `#minecraft:doors`        |            所有门            |
|     `#minecraft:trapdoors`      |            活板门            |
|       `#minecraft:fences`       |             栅栏             |
|    `#minecraft:fence_gates`     |            栅栏门            |
|      `#minecraft:flowers`       |           所有花朵           |
|       `#minecraft:crops`        | 可生长作物（小麦、胡萝卜等） |
|    `#minecraft:replaceable`     |   可被流体或生成替换的方块   |
|     `#minecraft:cauldrons`      |          各类炼药锅          |
|        `#minecraft:fire`        |           火类方块           |
|        `#minecraft:sand`        |       沙子类（含红沙）       |
|        `#minecraft:ice`         |           冰类方块           |
|    `#minecraft:valid_spawn`     |       可生成生物的方块       |

## 自定义标签

**原版存在许多标签,可以在原版标签的基础上扩展；也可以自定义标签**

### 扩展原版标签

- 原版标签均需要定义在`minecraft`命名空间下
- 直接在`minecraft`文件夹下的同样路径添加同名原版标签，并将`replace`字段设置为`false`即可扩展原版标签

### 自定义新标签

- 确定标签的命名空间，类型与路径，在对应位置创建标签文件即可
- 想要使用自定义标签时，依据命名空间，类型与路径创建`TagKey<T>`对象即可,泛型即为标签类型。

## 使用标签	

## 相关类

### `TagKey<T>`

> 表示一个标签文件,格式为`<namespace>:<type>/path`,如果有多层路径通过`/`分隔。

- 映射到真实标签文件位置即`data/tags/<namespace>/<type>/path`

### `BlockTags`

> 将原版存在的所有方块标签封装为了静态字段

### `ItemTags`

> 将原版存在的所有物品标签封装为了静态字段

# 配方

- 配方配置在`data/<namespace>/recipe/`
- 配方配置文件为`json`格式,名字与对应的物品一致

```java
{
  "type": "minecraft:crafting_shaped",
  "category": "misc",
  "group": "sticks",
  "key": {
    "#": "#minecraft:planks"
  },
  "pattern": [
    "#",
    "#"
  ],
  "result": {
    "count": 4,
    "id": "minecraft:stick"
  }
}
```

- `key`字段中的子字段如果值为标签,则格式为`#<namespace>:<tag_path>`

# 渲染

# 事件

## 自定义事件

# Mixin

> Fabric 生态系统中强大重要的工具，主要功能是通过注入自定义的逻辑、移除机制或者修改值，修改已经存在的代码

# 小知识

## 下载MC源码

> Fabric提供了gradle命令，用于生成MC源码供阅读

```
gradle genSources
```

## MC中的时间

> MC中的时间单位为Tick

<h4 align="center">1 Tick = 1/20秒</h4>

## 热更改

> 可以无需重启MC，热更新一些类或资源

**热更改类**

> 以调试模式启动Minecraft,在更改代码后，通过gradle的`classes`或`build`命令更新

- 只允许更改方法主体
- 很多注册相关操作只在启动时进行，即使更改了注册逻辑也无法重新注册

**热更改资源**

> 如纹理或模型，以调试模式启动后，重建项目并按 `F3 + T` 以应用更改

## 注册

Mc中所有东西均需要注册，无论哪种注册方法，最终底层都会转化为三个东西

Registry.register(注册表类型,注册键，实体对象)

- **注册表类型（Registry）**：告诉 Minecraft 你要注册的对象属于哪类，比如 `Registries.ITEM`、`Registries.BLOCK`、`Registries.ITEM_GROUP` 等。

- **注册键（RegistryKey / Identifier）**：全局唯一标识符，确保不同 MOD 或原版对象不会冲突。

- **实体对象**：你实际创建的对象，比如 `Item`、`Block` 或 `FabricItemGroup`。

## 资源获取

**相关网站**

- Minecraft Textures 官方资源

  ```
  https://mcasset.cloud/
  ```

  - 收录了**所有 Minecraft 各版本的原版资源**

- Planet Minecraft

  ```
  https://www.planetminecraft.com/
  ```

  - 有大量用户上传的**资源包（Resource Packs）**

**制作**

| 软件                        | 适用场景         | 特点                                               |
| --------------------------- | ---------------- | -------------------------------------------------- |
| **`Aseprite`**（推荐）      | 像素画           | 专业像素风工具，支持动画，有调色板和像素网格       |
| **`Piskel`**（网页版）      | 快速像素绘制     | 免费在线绘制像素图，https://www.piskelapp.com/     |
| **`GIMP`**                  | 通用绘图         | 免费开源，可编辑 PNG，功能接近 Photoshop           |
| **`Photopea`**（网页版 PS） | 快速编辑现有纹理 | https://www.photopea.com/                          |
| **`Blockbench`**            | 方块/模型贴图    | Minecraft 官方推荐建模软件，可以为模型直接绘制贴图 |

  

## 我的世界架构

> 我的世界是客户端-服务端架构（Client–Server Architecture）

| 部分                 | 功能               | 举例                                                       |
| -------------------- | ------------------ | ---------------------------------------------------------- |
| **Server（逻辑端）** | 游戏规则与数据权威 | 生物AI、方块更新、攻击判定、掉落物计算、物品使用、世界存储 |
| **Client（表现端）** | 渲染与交互         | 显示画面、播放声音、动画、粒子、输入响应                   |

