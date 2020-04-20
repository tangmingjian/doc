### AbstractAutoProxyCreator 

* 动态代理类的基类，用来创建动态代理对象

* 实现了BeanPostProcessor#postProcessAfterInitialization

* 执行时机:在InitializingBean#afterPropertiesSet()方法之后



```java
    @Override
    	public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
    		if (bean != null) {
    			Object cacheKey = getCacheKey(bean.getClass(), beanName);
    			//之前创建过，直接跳过
    			if (!this.earlyProxyReferences.contains(cacheKey)) {
    			    //创建代理
    				return wrapIfNecessary(bean, beanName, cacheKey);
    			}
    		}
    		return bean;
    	}

        protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {
    		if (beanName != null && this.targetSourcedBeans.contains(beanName)) {
    			return bean;
    		}
    		if (Boolean.FALSE.equals(this.advisedBeans.get(cacheKey))) {
    			return bean;
    		}
    		if (isInfrastructureClass(bean.getClass()) || shouldSkip(bean.getClass(), beanName)) {
    			this.advisedBeans.put(cacheKey, Boolean.FALSE);
    			return bean;
    		}
    
    		// Create proxy if we have advice.
    		//获取所有的advisors
    		Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, null);
    		if (specificInterceptors != DO_NOT_PROXY) {
    		    //有相关的advisors则开始创建代理
    			this.advisedBeans.put(cacheKey, Boolean.TRUE);
    			Object proxy = createProxy(
    					bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));
    			this.proxyTypes.put(cacheKey, proxy.getClass());
    			return proxy;
    		}
    
    		this.advisedBeans.put(cacheKey, Boolean.FALSE);
    		return bean;
    	}
```


* 自定义aspect切面，包括spring自带(@Transactional...)都会经过AbstractAutoProxyCreator处理
