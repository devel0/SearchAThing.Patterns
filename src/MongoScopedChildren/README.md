# MongoConcurrency

## Description

Some patterns to allow keep track of changed fields.

## Example db structure

![img](images/0_structure.PNG)

The collection contains DocumentEntity types, which inside contains two simple properties A, B plus one subitem of type NestedDocumentEntity and an Observablecollection of the same type.

## (1) Load

Here the same record of the collection is loaded twice with two separate connections.
First column Entity1 object.
Second column Entity2 object.
Third column after display current db content after each opeation. 

![img](images/1_load.PNG)

## (2) Modify

```
Entity1.A = a1
Entity1.Nested.C = c1
Entity1.Children.First().C = cc1

Entity2.B = b2
Entity2.Nested.D = d2
Entity2.Children.First().D = dd2
```

Here intentionally only not overlapped fields are updated, the intent is to see after save of Entity1 and then of Entity2 that last save of Entity2 not write its unmodified fields in order to avoid to overwrite changes done from Entity1 save.

![img](images/2_modify.PNG)

## (3) Save - Entity1

Entity1 saved

![img](images/3_save.PNG)

## (4) Save - Entity2

Entity2 save

![img](images/4_save.PNG)

## Pattern rules
- foreach object you want to keep tracking change of fields
    - implements `IMongoEntityTrackChanges` interface:
    ```csharp
    #region IMongoEntityTrackChanges
    HashSet<string> _ChangedProperties;
    public HashSet<string> ChangedProperties { get { return _ChangedProperties; } }
    #endregion
    ```

    - when not null add the changed property name to the ChangedProperties hashset. In fact this will be null during the loading phase in order to avoid auto-change:
    ```csharp
    ChangedProperties?.Add("A"); // use of ? operator ( until endinit is null )
    ```

    - implements `ISupportInitialize` interface. This way allow to allocate ChangedProperties hashset to be used when properties will be changed:
    ```csharp
    #region ISupportInitialize
    public void BeginInit()
    {
    }

    public void EndInit()
    {
        _ChangedProperties = new HashSet<string>();
    }
    #endregion
    ```

    - save changes using the UpdateDefinition array from the Changes extension method. (Note: This will clear the ChangedProperties)
    ```csharp
    repo.Update(Entity1, Entity1.Changes(repo.Updater));
    ```