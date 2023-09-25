Create New Product를 해보자
![[Pasted image 20230925134244.png]]

하단 박스에 옹기종기 모여있는 애들은 액션메뉴라고 하며 
찾아가다 보면 다음 파일에서 찾을 수 있다.
`widget/catalog/CatalogMenus.xml`
```xml
<menus xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://ofbiz.apache.org/Widget-Menu" xsi:schemaLocation="http://ofbiz.apache.org/Widget-Menu http://ofbiz.apache.org/dtds/widget-menu.xsd">
	...
	<menu name="MainActionMenu" menu-container-style="button-bar button-style-2" default-selected-style="selected">
        <menu-item name="newProduct" title="${uiLabelMap.ProductCreateNewProduct}">
            <condition>
                <and>
                    <or>
                        <if-has-permission permission="CATALOG" action="_CREATE"/>
                    </or>
                </and>
            </condition>
            <link target="EditProduct"/>
        </menu-item>
        <menu-item name="newStore" title="${uiLabelMap.ProductCreateNewProductStore}">
            <condition>
                <and>
                    <or>
                        <if-has-permission permission="CATALOG" action="_CREATE"/>
                    </or>
                </and>
            </condition>
            <link target="EditProductStore"/>
        </menu-item>
        
        <menu-item name="newCatalog" title="${uiLabelMap.ProductCreateNewCatalog}">
            <condition>
                <and>
                    <or>
                        <if-has-permission permission="CATALOG" action="_CREATE"/>
                    </or>
                </and>
            </condition>
            <link target="EditProdCatalog"/>
        </menu-item>
        <menu-item name="newCategory" title="${uiLabelMap.ProductCreateNewCategory}">
            <condition>
                <and>
                    <or>
                        <if-has-permission permission="CATALOG" action="_CREATE"/>
                    </or>
                </and>
            </condition>
            <link target="EditCategory"/>
        </menu-item>
    </menu>
    ...
</menus>
```

```xml
<menu-item name="newProduct" title="${uiLabelMap.ProductCreateNewProduct}">
	<condition>
		<and>
			<or>
				<if-has-permission permission="CATALOG" action="_CREATE"/>
			</or>
		</and>
	</condition>
	<link target="EditProduct"/>
</menu-item>
```
하나만 분석해보자면, `<condition>`에 따라 display 여부를 결정하는 것 같으며
`<if-has-permission permission="CATALOG" action="_CREATE"/>` 는 해당 유저가 CATALOG라는 자원에 CREATE 권한이 있는지를 뜻한다.




