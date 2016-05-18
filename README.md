


##Java反射常用工具类

俗话说的好：“无反射，无框架！”
最近由于频繁的对客户端打patch，无时无刻不用到反射，竟然发现原来写的反射工具类有bug。
总结一下使用反射的常见使用，本人语言组织能力极差，以下尽可能用代码说话
------------关于反射调用方法--------------
1. 获取方法Method对象
    - Class.getMethod()此方法只能获取到public的方法
    - Class.getDeclaredMethod()此方法可以获取到所有的方法，没有修饰符的影响
    注意：从效率上讲，getDeclaredMethod()方法的效率是比getMethod()低的，因此如果知道需要获取的Method的修饰符，最好选择效率高的方法。
    通用的方法：
    
    ```
        /**
         * 获取某个类声明的Method对象，会依次寻找当前类和其父类，找到则返回
         */
        public static Method getDeclaredMethod(Class<?> clazz, String methodName,
                                               Class<?>... parameterTypes) {
            if(clazz != null){
                //从当前类开始寻找，依次想其父类开始寻找，知道找到为止
                for (; Object.class != clazz; clazz = clazz.getSuperclass()){
                    try {
                        Method declaredMethod = clazz.getDeclaredMethod(methodName, parameterTypes);
                        if(declaredMethod != null){
                            declaredMethod.setAccessible(true);
                            return declaredMethod;
                        }
                    } catch (NoSuchMethodException e) {
                        e.printStackTrace();
                    }
                }
            }
            return null;
        }
    ```

2. 获取某个类的父类的方法Mehod对象：不需要过多解释了吧，看懂上面的你肯定就会这个了。
    ```
        /**
         * 获取父类的方法
         */
        public static Method getSuperMethod(Class<?> currentClazz, String methodName, Class<?>... parameterTypes){
            if(currentClazz != null && currentClazz.getSuperclass() != Object.class){
    
                for(currentClazz = currentClazz.getSuperclass(); Object.class != currentClazz ; currentClazz = currentClazz.getSuperclass()){
                    try {
                        Method declaredMethod = currentClazz.getDeclaredMethod(methodName, parameterTypes);
                        if(declaredMethod != null){
                            declaredMethod.setAccessible(true);
                            return declaredMethod;
                        }
                    } catch (NoSuchMethodException e) {
                        e.printStackTrace();
                    }
    
                }
            }
            return null;
        }
    ```


3. 调用super方法，即调用父类的方法
    ```
        /**
         * 调用当前对象父类的方法，即super方法
         */
        public static Object invokeSuperMethod(Object currentObject, String methodName, Class<?>... parameterTypes) throws Exception {
            if(currentObject != null){
                Method superMethod = getSuperMethod(currentObject.getClass(), methodName, parameterTypes);
                if(superMethod != null){
                    superMethod.setAccessible(true);
                    return  superMethod.invoke(currentObject, parameterTypes);
                }
            }
            throw new Exception("invoke method fail");
        }
    ```
4. 调用方法
    ```
        /**
     *调用当前对象的方法，可用于静态和非静态方法
     */
    public static Object invokeMethod(Object object, String methodName,
                                      Class<?>[] parameterTypes, Object[] paramValues) throws Exception {
        Method method = getDeclaredMethod(object.getClass(), methodName,
                parameterTypes);
        if (method != null) {
            return method.invoke(object, paramValues);
        }
        throw new Exception("invoke method fail");
    }
    ```

5. 调用静态方法(private、public或protected)
    调用时直接使用invoke(null, parameterTypes)即可，当然此处使用一个对象也没有问题, API的解释为`If the underlying method is static, then the specified obj argument is ignored. It may be null.`
    
    ```
    /**
     * 调用静态方法
     */
    public static Object invokeStaticMethod(Object object, String methodName,
                                      Class<?>[] parameterTypes, Object[] paramValues) throws Exception {
        if(object != null){
            Method method = getDeclaredMethod(object.getClass(), methodName,
                    parameterTypes);
            if (method != null) {
                method.setAccessible(true);
                return method.invoke(null, paramValues);
            }
        }
        throw new Exception("invoke method fail");
    }    
    ```
    
------------------关于字段----------------------

1. 获取字段
    ```
        private static Field getDeclaredField(Class<?> clazz, String fieldName) {
            if (clazz != null) {
                StringBuffer stringBuffer = new StringBuffer();
                stringBuffer.append(clazz.getName());
                stringBuffer.append(fieldName);
                String key = stringBuffer.toString();
                if (fieldCache.containsKey(key)) {
                    return fieldCache.get(key);
                } else {
                    Field field = null;
                    for (; clazz != Object.class; clazz = clazz.getSuperclass()) {
                        try {
                            field = clazz.getDeclaredField(fieldName);
                            field.setAccessible(true);
                            fieldCache.put(key, field);
                            return field;
                        } catch (Exception e) {
                        }
                    }
                }
            }
            return null;
        }
    ```

2. 获取某个对象的某个字段的值
    ```
        /**
         * 根据Object对象获取Object中的fieldValue，一般用作获取非静态字段
         * @param object
         * @param fieldName
         * @return
         * @throws Exception
         */
        public static Object getFieldValue(Object object, String fieldName)
                throws Exception {
            Field field = getDeclaredField(object.getClass(), fieldName);
            if (field != null) {
                return field.get(object);
            } else {
                throw new Exception("getFieldValue fail");
            }
    
        }
    ```
3. 给某个对象的某个字段设置value    
    ```
        public static void setFieldValue(Class<?> clazz, String fieldName,
                                         Object value) throws Exception {
            Field field = getDeclaredField(clazz, fieldName);
            if (field != null) {
                field.set(null, value);
            } else {
                throw new Exception(fieldName+" fail");
            }
        }
    ``` 


