
[https://localhost:8443/catalog/control/FindProduct](https://localhost:8443/catalog/control/FindProduct)

로의 path resolve 과정을 알아보자..

`/applications/component-load.xml`

```xml
<component-loader xmlns:xsi="<http://www.w3.org/2001/XMLSchema-instance>"
        xsi:noNamespaceSchemaLocation="<https://ofbiz.apache.org/dtds/component-loader.xsd>">
    <load-component component-location="datamodel"/>
    <load-component component-location="party"/>
    <load-component component-location="securityext"/>
    <load-component component-location="content"/>
    <load-component component-location="workeffort"/>
    <load-component component-location="product"/>
    <load-component component-location="manufacturing"/>
    <load-component component-location="accounting"/>
    <load-component component-location="humanres"/>
    <load-component component-location="order"/>
    <load-component component-location="marketing"/>
<!-- common component used by most other components last because it uses info from most components-->
    <load-component component-location="commonext"/>
</component-loader>
```

에서 로드할 컴포넌트 등록.

`/applications/product/ofbiz-component.xml`

```xml
<!-- 컴포넌트 이름 요잉네 -->
<ofbiz-component name="product"
        xmlns:xsi="<http://www.w3.org/2001/XMLSchema-instance>"
        xsi:noNamespaceSchemaLocation="<https://ofbiz.apache.org/dtds/ofbiz-component.xsd>">
	<resource-loader name="main" type="component"/>
  <classpath type="dir" location="config"/>

	<!-- 서비스 리소스랑 엔티티 리소스들 요다 등록 -->
	<entity-resource type="data" reader-name="seed" loader="main" location="data/ProductSecurityPermissionSeedData.xml"/>
  ...
	<service-resource type="model" loader="main" location="servicedef/services.xml"/>
	...
	<!--공통 서비스 개념인 Product로 묶인 거라, 이런식으로 웹앱도 각자 등록함, 
		각각의 디렉토리, mount-point를 가짐-->
	<webapp name="catalog"
        title="Catalog"
        position="2"
        description="CatalogComponentDescription"
        server="default-server"
        location="webapp/catalog"
        base-permission="OFBTOOLS,CATALOG"
        app-shortcut-screen="component://product/widget/catalog/CatalogScreens.xml#ShortcutApp"
        mount-point="/catalog"/>
    <webapp name="facility"
        title="Facility"
        position="3"
        description="FacilityComponentDescription"
        server="default-server"
        location="webapp/facility"
        base-permission="OFBTOOLS,FACILITY"
        app-shortcut-screen="component://product/widget/facility/FacilityScreens.xml#ShortcutApp"
        mount-point="/facility"/>
</ofbiz-component>
```

위 파일의 xsd는

`/framework/base/dtd/ofbiz-component.xsd`

```xml
<xs:schema xmlns:xs="<http://www.w3.org/2001/XMLSchema>" elementFormDefault="qualified">
	<xs:element name="service-resource">
      <xs:complexType>
          <xs:attributeGroup ref="attlist.service-resource"/>
      </xs:complexType>
  </xs:element>
  <xs:attributeGroup name="attlist.service-resource">
      <xs:attribute name="type" use="required">
          <xs:simpleType>
              <xs:restriction base="xs:token">
                  <xs:enumeration value="model"/>
                  <xs:enumeration value="group"/>
                  <xs:enumeration value="eca"/>
                  <xs:enumeration value="mca"/>
              </xs:restriction>
          </xs:simpleType>
      </xs:attribute>
      <xs:attribute type="xs:string" name="loader" use="required"/>
      <xs:attribute type="xs:string" name="location" use="required"/>
  </xs:attributeGroup>
	...
</xs:schema>
```

이런 식으로 돼있음.

xml 파일에 쓰인 엘리먼트들의 스키마가 나온다. 엔진은 어디 있는지 모르겠다.

아무튼

[https://localhost:8443/catalog/control/FindProduct](https://localhost:8443/catalog/control/FindProduct) 구조에서 mount-point가 “/catalog”인 웹앱의 location 속성이 “webapp/catalog/control…”이므로 webapp/catalog/WEB-INF/controller.xml로 이후 resolving을 이관한다. 거기서 request-map 엘리먼트의 uri가 “FindProduct”인걸 찾으면 되겠다.

`/applications/product/webapp/catalog/WEB-INF/controller.xml`

```xml
<site-conf xmlns:xsi="<http://www.w3.org/2001/XMLSchema-instance>"
        xmlns="<http://ofbiz.apache.org/Site-Conf>" xsi:schemaLocation="<http://ofbiz.apache.org/Site-Conf> <http://ofbiz.apache.org/dtds/site-conf.xsd>">
    <include location="component://common/webcommon/WEB-INF/common-controller.xml"/>
    <include location="component://common/webcommon/WEB-INF/portal-controller.xml"/>
    <include location="component://commonext/webapp/WEB-INF/controller.xml"/>
		<description>Catalog Module Site Configuration File</description>
		...
		<request-map uri="FindProduct">
        <security https="true" auth="true"/>
        <response name="success" type="view" value="FindProduct"/>
    </request-map>
		...
		<!--
			이 두개가 매칭됨
			component://product/widget/catalog/ProductScreens.xml#FindProduct를 찾아봐야 될거같은데,
			component://가 붙은건 ofbiz-framework/applications를 루트디렉토리로 가지는 것 같다.
		-->
		<view-map name="FindProduct" type="screen" page="component://product/widget/catalog/ProductScreens.xml#FindProduct"/>
</site-conf>
```

`/applications/product/widget/catalog/ProductScreens.xml`
```xml
<screens xmlns:xsi="<http://www.w3.org/2001/XMLSchema-instance>"
        xmlns="<http://ofbiz.apache.org/Widget-Screen>" xsi:schemaLocation="<http://ofbiz.apache.org/Widget-Screen> <http://ofbiz.apache.org/dtds/widget-screen.xsd>">
	<screen name="FindProduct">
      <section>
          <actions>
              <set field="titleProperty" value="PageTitleFindProduct"/>
          </actions>
          <widgets>
              <decorator-screen name="CommonProductDecorator" location="${parameters.productDecoratorLocation}">
                  <decorator-section name="body">
                      <section>
                          <widgets>
                              <decorator-screen name="FindScreenDecorator" location="component://common/widget/CommonScreens.xml">
                                  <decorator-section name="search-options">
                                      <include-form name="FindProduct" location="component://product/widget/catalog/ProductForms.xml"/>
                                  </decorator-section>
                                  <decorator-section name="search-results">
                                      <include-grid name="ListProducts" location="component://product/widget/catalog/ProductForms.xml"/>
                                  </decorator-section>
                              </decorator-screen>
                          </widgets>
                      </section>
                  </decorator-section>
              </decorator-screen>
          </widgets>
       </section>
  </screen>
	...
</screens>
```

드디어 위젯이 나온다.

`/applications/product/widget/catalog/ProductForms.xml`
```xml
<forms xmlns:xsi="<http://www.w3.org/2001/XMLSchema-instance>"
        xmlns="<http://ofbiz.apache.org/Widget-Form>" xsi:schemaLocation="<http://ofbiz.apache.org/Widget-Form> <http://ofbiz.apache.org/dtds/widget-form.xsd>">
    <form name="FindProduct" type="single" target="FindProduct" title="" default-map-name="product"
        header-row-style="header-row" default-table-style="basic-table">
        <field name="noConditionFind"><hidden value="Y"/><!-- if this isn't there then with all fields empty no query will be done --></field>
        <field name="productId" title="${uiLabelMap.ProductProductId}"><text-find/></field>
        <field name="internalName" title="${uiLabelMap.ProductInternalName}"><text-find/></field>
        <field name="submitButton" title="${uiLabelMap.CommonFind}" widget-style="smallSubmit">
            <submit button-type="button"/>
        </field>
    </form>
    <grid name="ListProducts" list-name="list" paginate-target="FindProduct" override-list-size="true"
        odd-row-style="alternate-row" default-table-style="basic-table hover-bar" header-row-style="header-row-2">
        <actions>
            <set field="entityName" value="Product"/>
            <service service-name="performFindList" result-map="result" result-map-list="list">
                <field-map field-name="inputFields" from-field="requestParameters"/>
                <field-map field-name="entityName" from-field="entityName"/>
                <field-map field-name="orderBy" from-field="parameters.sortField"/>
                <field-map field-name="viewIndex" from-field="viewIndex"/>
                <field-map field-name="viewSize" from-field="viewSize"/>
            </service>
        </actions>
        <field name="productId" sort-field="true" widget-style="buttontext">
            <hyperlink description="${productId}" target="EditProduct" also-hidden="false">
                <parameter param-name="productId"/>
            </hyperlink>
        </field>
        <field name="productTypeId" sort-field="true"><display-entity entity-name="ProductType"/></field>
        <field name="internalName" sort-field="true"><display/></field>
        <field name="brandName" sort-field="true"><display/></field>
        <field name="productName" sort-field="true"><display/></field>
        <field name="description" sort-field="true"><display/></field>
    </grid>
		...
</forms>
```

서비스를 찾아야되는데
`<form target="FindProduct">` 과 `<grid paginate-target="FindProduct">` 값을 맞춰주면 되는거 같다. 그러면 grid 내부의 `<actions>`가 폼 데이터를 받아서 작동하는듯?

performFindList라는 서비스는
`/framework/common/servicedef/services.xml` 여기있다.

이걸 어케찾음? 나도 모름. 
그냥 내가 만들 서비스들은 /applications/../servicedef에 작성하면 될 듯 하다.

```xml
<services xmlns:xsi="<http://www.w3.org/2001/XMLSchema-instance>"
        xsi:noNamespaceSchemaLocation="<https://ofbiz.apache.org/dtds/services.xsd>">
    <description>Common Application Services</description>
    <vendor>OFBiz</vendor>
    <version>1.0</version>
		...
	<service name="performFindList" auth="false" engine="java" invoke="performFindList" location="org.apache.ofbiz.common.FindServices">
      <description>Generic service to return an partial list.  set filterByDate to Y to exclude expired records.
          set noConditionFind to Y to find without conditions. 
          If used in a form, it is necessary to assign a value (true makes sense) to override-list-size attribute so that 
          FormRenderer.renderItemRows sets the lowIndex correctly, because once the results of performFindList are displayed, 
          otherwise pages > 0 are rendered as empty. see OFBIZ-6422 + 6423 for details</description>
      <attribute name="entityName" type="String" mode="IN" optional="false"/>
      <attribute name="inputFields" type="java.util.Map" mode="IN" optional="false"/>
      <attribute name="orderBy" type="String" mode="IN" optional="true"/>
      <attribute name="noConditionFind" type="String" mode="IN" optional="true"><!-- find with no condition (empty entityConditionList) only done when this is Y --></attribute>
      <attribute name="filterByDate" type="String" mode="IN" optional="true"/>
      <attribute name="filterByDateValue" type="Timestamp" mode="IN" optional="true"/>
      <attribute name="viewIndex" type="Integer" mode="IN" optional="true"/>
      <attribute name="viewSize" type="Integer" mode="IN" optional="true"/>
      <attribute name="list" type="List" mode="OUT" optional="true"/>
      <attribute name="listSize" type="Integer" mode="OUT" optional="false"/>
      <attribute name="queryString" type="String" mode="OUT" optional="true"/>
      <attribute name="queryStringMap" type="java.util.Map" mode="OUT" optional="true"/>
  </service>
	...
</services>
```


<set field="entityName" value="Product"/> 해 주고 서비스를 호출하면 알아서 그 엔티티에 맞는 리스트를 리턴하는듯 하다..
`\framework\common\src\main\java\org\apache\ofbiz\common\FindServices.java`
```java
public class FindServices {
	public static Map<String, Object> performFindList(DispatchContext dctx, Map<String, Object> context) {
      Integer viewSize = (Integer) context.get("viewSize");
      if (viewSize == null) {
          viewSize = 20;       // default
      }
      context.put("viewSize", viewSize);
      Integer viewIndex = (Integer) context.get("viewIndex");
      if (viewIndex == null) {
          viewIndex = 0;  // default
      }
      context.put("viewIndex", viewIndex);

      Map<String, Object> result = performFind(dctx, context);

      int start = viewIndex * viewSize;
      List<GenericValue> list = null;
      Integer listSize = 0;
      try (EntityListIterator it = (EntityListIterator) result.get("listIt")) {
          list = it.getPartialList(start + 1, viewSize); // list starts at '1'
          listSize = it.getResultsSizeAfterPartialList();
      } catch (ClassCastException | NullPointerException | GenericEntityException e) {
          Debug.logInfo("Problem getting partial list" + e, MODULE);
      }

      result.put("listSize", listSize);
      result.put("list", list);
      result.remove("listIt");
      return result;
  }
	public static Map<String, Object> performFind(DispatchContext dctx, Map<String, ?> context) {
        String entityName = (String) context.get("entityName");
        DynamicViewEntity dynamicViewEntity = (DynamicViewEntity) context.get("dynamicViewEntity");
        String orderBy = (String) context.get("orderBy");
        Map<String, ?> inputFields = checkMap(context.get("inputFields"), String.class, Object.class); // Input
        String noConditionFind = (String) context.get("noConditionFind");
        String distinct = (String) context.get("distinct");
        List<String> fieldList = UtilGenerics.cast(context.get("fieldList"));
        GenericValue userLogin = (GenericValue) context.get("userLogin");
        Locale locale = (Locale) context.get("locale");
        Delegator delegator = dctx.getDelegator();
        if (UtilValidate.isEmpty(noConditionFind)) {
            // try finding in inputFields Map
            noConditionFind = (String) inputFields.get("noConditionFind");
        }
        if (UtilValidate.isEmpty(noConditionFind)) {
            // Use configured default
            noConditionFind = EntityUtilProperties.getPropertyValue("widget", "widget.defaultNoConditionFind", delegator);
        }
        String filterByDate = (String) context.get("filterByDate");
        if (UtilValidate.isEmpty(filterByDate)) {
            // try finding in inputFields Map
            filterByDate = (String) inputFields.get("filterByDate");
        }
        Timestamp filterByDateValue = (Timestamp) context.get("filterByDateValue");
        String fromDateName = (String) context.get("fromDateName");
        if (UtilValidate.isEmpty(fromDateName)) {
            // try finding in inputFields Map
            fromDateName = (String) inputFields.get("fromDateName");
        }
        String thruDateName = (String) context.get("thruDateName");
        if (UtilValidate.isEmpty(thruDateName)) {
            // try finding in inputFields Map
            thruDateName = (String) inputFields.get("thruDateName");
        }

        Integer viewSize = (Integer) context.get("viewSize");
        Integer viewIndex = (Integer) context.get("viewIndex");
        Integer maxRows = null;
        if (viewSize != null && viewIndex != null) {
            maxRows = viewSize * (viewIndex + 1);
        }

        LocalDispatcher dispatcher = dctx.getDispatcher();

        Map<String, Object> prepareResult = null;
        try {
            prepareResult = dispatcher.runSync("prepareFind", UtilMisc.toMap("entityName", entityName, "orderBy", orderBy,
                                               "dynamicViewEntity", dynamicViewEntity,
                                               "inputFields", inputFields, "filterByDate", filterByDate, "noConditionFind", noConditionFind,
                                               "filterByDateValue", filterByDateValue, "userLogin", userLogin, "fromDateName", fromDateName,
                    "thruDateName", thruDateName,
                                               "locale", context.get("locale"), "timeZone", context.get("timeZone")));
        } catch (GenericServiceException gse) {
            return ServiceUtil.returnError(UtilProperties.getMessage(RESOURCE, "CommonFindErrorPreparingConditions",
                    UtilMisc.toMap("errorString", gse.getMessage()), locale));
        }
        EntityConditionList<EntityCondition> exprList = UtilGenerics.cast(prepareResult.get("entityConditionList"));
        List<String> orderByList = checkCollection(prepareResult.get("orderByList"), String.class);

        Map<String, Object> executeResult = null;
        try {
            executeResult = dispatcher.runSync("executeFind", UtilMisc.toMap("entityName", entityName, "orderByList", orderByList,
                                                                             "dynamicViewEntity", dynamicViewEntity,
                                                                             "fieldList", fieldList, "entityConditionList", exprList,
                                                                             "noConditionFind", noConditionFind, "distinct", distinct,
                                                                             "locale", context.get("locale"), "timeZone", context.get("timeZone"),
                                                                             "maxRows", maxRows));
        } catch (GenericServiceException gse) {
            return ServiceUtil.returnError(UtilProperties.getMessage(RESOURCE, "CommonFindErrorRetrieveIterator",
                    UtilMisc.toMap("errorString", gse.getMessage()), locale));
        }

        if (executeResult.get("listIt") == null) {
            if (Debug.verboseOn()) {
                Debug.logVerbose("No list iterator found for query string + [" + prepareResult.get("queryString") + "]", MODULE);
            }
        }

        Map<String, Object> results = ServiceUtil.returnSuccess();
        results.put("listIt", executeResult.get("listIt"));
        results.put("listSize", executeResult.get("listSize"));
        results.put("queryString", prepareResult.get("queryString"));
        results.put("queryStringMap", prepareResult.get("queryStringMap"));
        return results;
    }
	
}
```
