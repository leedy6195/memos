`WEB-INF/controller.xml`
```xml
<request-map uri="test">  
    <security https="true"/>  
    <event type="java" path="org.apache.ofbiz.order.order.OrderServices" invoke="test"/>  
    <response name="success" type="request" value="json"/>  
    <response name="error" type="request" value="json"/>  
</request-map>

```



`OrderServices.java`
```Java
public static String test(HttpServletRequest request, HttpServletResponse response) {  
    String orderId = request.getParameter("orderId");  
    request.setAttribute("test", "done");  
    EntityCondition condition = EntityCondition.makeCondition("orderId", EntityOperator.EQUALS, orderId);  
    Delegator delegator = (Delegator) request.getAttribute("delegator");  
  
    try {  
        //GenericValue orderItem = delegator.findOne("OrderItem", UtilMisc.toMap("orderId", orderId), false);  
        List<GenericValue> orderItems = delegator.findList("OrderItem", condition, null, null, null, false);  
        if (orderItems != null && !orderItems.isEmpty()) {  
            GenericValue orderItem = orderItems.get(0);  
            if (orderItem != null) {  
                boolean isPromo = Objects.equals(orderItem.getString("isPromo"), "Y");  
                request.setAttribute("isPromo", isPromo);  
            }  
        } else {  
            request.setAttribute("isPromo", null);  
        }  
    } catch (GenericEntityException e) {  
        e.printStackTrace();  
    }  
    return "success";  
}


```

