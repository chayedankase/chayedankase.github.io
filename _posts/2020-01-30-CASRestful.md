---
layout: post
title: 通过Rest接口实现CAS单点登录
categories: [单点登录]
description: restful接口单点登录，允许接入系统自定义登录页面
keywords: restful,cas
---



## 通过Rest接口实现CAS单点登录

> 打算做客户端通过rest接口来进行单点登录，这样可以方便的让任意客户端来自定义登录页面的样式等。
>
> 通过官网的说明，可以获取到TGT，通过TGT获取到ST，可以实现服务访问，但是登录成功后，访问其他的SSO系统，由于server域中并没有TGC的cooki保存，因而无法实现登录。
>
> 初步设想是rest登录成功后，附带地址参数重定向至服务端，让服务端进行了cooki保存，后续其他的SSO系统就可以照常接入了。

#### 1.导入依赖

> 不导入依赖的话源码看不到，overlays的这种方式确实挺恶心
>
> ```xml-dtd
> <!--开启rest认证-->
> <dependency>
>     <groupId>org.apereo.cas</groupId>
>     <artifactId>cas-server-support-rest</artifactId>
>     <version>${cas.version}</version>
> </dependency>
> <!--自定义效验的依赖-->
> <dependency>
>     <groupId>org.apereo.cas</groupId>
>     <artifactId>cas-server-core-authentication</artifactId>
>     <version>${cas.version}</version>
> </dependency>
> <dependency>
>     <groupId>org.apereo.cas</groupId>
>     <artifactId>cas-server-core-authentication-api</artifactId>
>     <version>${cas.version}</version>
> </dependency>
> <dependency>
>     <groupId>org.apereo.cas</groupId>
>     <artifactId>cas-server-core-webflow</artifactId>
>     <version>${cas.version}</version>
> </dependency>
> <dependency>
>     <groupId>org.apereo.cas</groupId>
>     <artifactId>cas-server-core-webflow-api</artifactId>
>     <version>${cas.version}</version>
> </dependency>
> <!--登陆时的效验-->
> <dependency>
>     <groupId>org.apereo.cas</groupId>
>     <artifactId>cas-server-core-cookie</artifactId>
>     <version>${cas.version}</version>
> </dependency>
> <dependency>
>     <groupId>org.apereo.cas</groupId>
>     <artifactId>cas-server-support-actions</artifactId>
>     <version>${cas.version}</version>
> </dependency>
> ```

#### 2. 启动loginflow分析

> cas登录使用了webflow, 所以先去webflow的相关包看看。
>
> ![image-20200130144602011]({{site.url}}\imgs\cas\2\image-20200130144602011.png)
>
> 找到如图的casWebflowConfigurer，webflow的配置接口。
>
> ```JAVA
> /**
>  * This is {@link CasWebflowConfigurer}.
>  *
>  * @author Misagh Moayyed
>  * @since 5.0.0
>  */
> public interface CasWebflowConfigurer extends Ordered {
> 
>     /**
>      * Main login flow id.
>      */
>     String FLOW_ID_LOGIN = "login";
> 
>     /**
>      * Main logout flow id.
>      */
>     String FLOW_ID_LOGOUT = "logout";
> ```
>
> 找找实现类，发现 DefaultLoginWebflowConfigurer，应该就是这玩意，进去看看。
>
> ![image-20200130144850144]({{site.url}}\imgs\cas\2\image-20200130144850144.png)
>
> 存在一个如下的初始化方法,debug看看。感觉这里每一步都值得详细看看。
>
> ```java
>  @Override
>     protected void doInitialize() {
>         //1.获取加载login-webflow.xml后的webflow
>         final Flow flow = getLoginFlow();
>         if (flow != null) {
>             //2.给flow中添加初始化的FlowActions
>             createInitialFlowActions(flow);
>             //3.创建全局异常处理的节点
>             createDefaultGlobalExceptionHandlers(flow);
>             //4.创建登录结束的节点
>             createDefaultEndStates(flow);
>             //5.添加描述节点
>             createDefaultDecisionStates(flow);
>             //6.添加执行节点
>             createDefaultActionStates(flow);
>             createDefaultViewStates(flow);
>             createRememberMeAuthnWebflowConfig(flow);
>             setStartState(flow, CasWebflowConstants.STATE_ID_INITIAL_AUTHN_REQUEST_VALIDATION_CHECK);
>         }
>     }
> ```
>

##### 1. 加载login-webflow.xml

> ```java
> final Flow flow = getLoginFlow();
> 
> @Override
>     public Flow getLoginFlow() {
>         if (this.loginFlowDefinitionRegistry == null) {
>             LOGGER.error("Login flow registry is not configured and/or initialized correctly.");
>             return null;
>         }
>         final boolean found = Arrays.stream(this.loginFlowDefinitionRegistry.getFlowDefinitionIds()).anyMatch(f -> f.equals(FLOW_ID_LOGIN));
>         if (found) {
>             return (Flow) this.loginFlowDefinitionRegistry.getFlowDefinition(FLOW_ID_LOGIN);
>         }
>         LOGGER.error("Could not find flow definition [{}]. Available flow definition ids are [{}]", FLOW_ID_LOGIN, this.loginFlowDefinitionRegistry.getFlowDefinitionIds());
>         return null;
>     }
> ```
>
> 遍历loginFlowDefinitionRegistry中的flow,如果有存在名称为 FLOW_ID_LOGIN 也就是 “login”的，就将这个flow返回，否则返回null。

##### 2.添加初始化FlowActions

> ​	createInitialFlowActions(flow);
>
> ```java
> protected void createInitialFlowActions(final Flow flow) {
>         flow.getStartActionList().add(createEvaluateAction(CasWebflowConstants.ACTION_ID_INIT_FLOW_SETUP));
>     }
> ```
>
> 创建了一个名称为ACTION_ID_INIT_FLOW_SETUP的Action,作为初始化action,即在调用login webflow前会先执行，这个Action的 doExecute(final RequestContext context)方法。
>
>  org.apereo.cas.web.flow.login.InitialFlowSetupAction，这里后面再细说。
>
> ```java
>  @Override
>     public Event doExecute(final RequestContext context) {
>         configureCookieGenerators(context);
>         configureWebflowContext(context);
>         configureWebflowContextForService(context);
>         return success();
>     }
> ```

##### 3. 创建全局异常处理

> createDefaultGlobalExceptionHandlers(flow);
>
> ```java
> protected void createDefaultGlobalExceptionHandlers(final Flow flow) {
>         final TransitionExecutingFlowExecutionExceptionHandler h = new TransitionExecutingFlowExecutionExceptionHandler();
>         h.add(UnauthorizedSsoServiceException.class, CasWebflowConstants.STATE_ID_VIEW_LOGIN_FORM);
>         h.add(NoSuchFlowExecutionException.class, CasWebflowConstants.STATE_ID_VIEW_SERVICE_ERROR);
>         h.add(UnauthorizedServiceException.class, CasWebflowConstants.STATE_ID_SERVICE_UNAUTHZ_CHECK);
>         h.add(UnauthorizedServiceForPrincipalException.class, CasWebflowConstants.STATE_ID_SERVICE_UNAUTHZ_CHECK);
>         h.add(PrincipalException.class, CasWebflowConstants.STATE_ID_SERVICE_UNAUTHZ_CHECK);
>         flow.getExceptionHandlerSet().add(h);
>     }
> ```
>
> 不同的异常进入不同的节点。

### SSO访问认证流程

> 1. 客户端访问资源未登录，携带访问路径，重定向到认证中心。
> 2. 认证中心从cookie和请求头中获取名为 tgc的认证令牌。
> 3. 将获取到的tgc令牌拆解为tgt、ip、浏览器信息3部分（默认情况下）。
> 4. 把tgt分别存入request域 、flow域、session域中（如果session域中没有的话）

##### 1. 客户端登录

> 客户端重定向到认证中心，填写用户密码，点击登录。
>
> 认证中心的受限服务都会先被拦截验证是否有cookie
>
> org.apereo.cas.web.support.CookieRetrievingCookieGenerator 应该是它的子类
>
> org.apereo.cas.web.support.TGCCookieRetrievingCookieGenerator.retrieveCookieValue( request)
>
> ```java
> /**
>      *这里是从cookie或请求头中获取TGC，获取不到返回null,如果获取到了，则将TGC解析
>      *为TGT返回。
>      */
>     public String retrieveCookieValue(final HttpServletRequest request) {
>         try {
>             Cookie cookie = org.springframework.web.util.WebUtils.getCookie(request, getCookieName());
>             if (cookie == null) {
>                 final String cookieValue = request.getHeader(getCookieName());
>                 if (StringUtils.isNotBlank(cookieValue)) {
>                     LOGGER.debug("Found cookie [{}] under header name [{}]", cookieValue, getCookieName());
>                     cookie = createCookie(cookieValue);
>                 }
>             }
> 
>             return cookie == null ? null : this.casCookieValueManager.obtainCookieValue(cookie, request);
>         } catch (final Exception e) {
>             LOGGER.debug(e.getMessage(), e);
>         }
>         return null;
>     }
> ```
>
> 此处返回null,表示未登录，则进入身份认证阶段。

##### 2. 认证阶段

> 通过login-webflow.xml中可以看出，当进行提交操作后，则会进行如下操作
>
> org.apereo.cas.web.flow.actions.AbstractAuthenticationAction.doExecute(requestContext)
>
> 实际是子类 InitialAuthenticationAction。
>
> ```java
> @Override
>     protected Event doExecute(final RequestContext requestContext) {
>         //获取浏览器信息
>         final String agent = WebUtils.getHttpServletRequestUserAgentFromRequestContext();
>         //获取请求地理信息
>         final GeoLocationRequest geoLocation = WebUtils.getHttpServletRequestGeoLocationFromRequestContext();
> 		//判断请求的ip
>         if (geoLocation != null && StringUtils.isNotBlank(agent) && !adaptiveAuthenticationPolicy.apply(agent, geoLocation)) {
>             final String msg = "Adaptive authentication policy does not allow this request for " + agent + " and " + geoLocation;
>             final Map<String, Throwable> map = CollectionUtils.wrap(UnauthorizedAuthenticationException.class.getSimpleName(), new UnauthorizedAuthenticationException(msg));
>             final AuthenticationException error = new AuthenticationException(msg, map, new HashMap<>(0));
>             return new Event(this, CasWebflowConstants.TRANSITION_ID_AUTHENTICATION_FAILURE,
>                 new LocalAttributeMap(CasWebflowConstants.TRANSITION_ID_ERROR, error));
>         }
> 
>         final Event serviceTicketEvent = this.serviceTicketRequestWebflowEventResolver.resolveSingle(requestContext);
>         if (serviceTicketEvent != null) {
>             fireEventHooks(serviceTicketEvent, requestContext);
>             return serviceTicketEvent;
>         }
> //这里会调用认证的handler,执行认证逻辑，如果登录成功则返回success.
>         final Event finalEvent = this.initialAuthenticationAttemptWebflowEventResolver.resolveSingle(requestContext);
>         fireEventHooks(finalEvent, requestContext);
>         return finalEvent;
>     }
> ```
>
> 1. 从request中获取浏览器的信息 agent。
>
> 2. 获取地理信息。
>
> 3. 判断agent与地理信息及ip是否允许访问,有用的就是配置个ip过滤。
>
>    adaptiveAuthenticationPolicy.apply(agent, geoLocation)
>
>    ```java
>    @Override
>        public boolean apply(final String userAgent, final GeoLocationRequest location) {
>            //获取客户端的信息，里面包含了客户端的ip和服务端的ip
>            final ClientInfo clientInfo = ClientInfoHolder.getClientInfo();
>            //信息未获取到则不进行判断，直接返回true.
>            if (clientInfo == null || StringUtils.isBlank(userAgent)) {
>                return true;
>            }
>            //获取客户端ip，判断ip是否是被拒绝的。这里的被拒绝可以去配置。
>            final String clientIp = clientInfo.getClientIpAddress();
>            if (isClientIpAddressRejected(clientIp)) {
>                //跟进去可以看到
>                return false;
>            }
>            if (isUserAgentRejected(userAgent)) {
>                return false;
>            }
>            //对地理位置进行判断，这里没有地理位置服务，所以无法判断。
>            if (this.geoLocationService != null && location != null && StringUtils.isNotBlank(clientIp)
>                && StringUtils.isNotBlank(this.adaptiveAuthenticationProperties.getRejectCountries())) {
>                final GeoLocationResponse loc = this.geoLocationService.locate(clientIp, location);
>                if (loc != null) {
>                    if (isGeoLocationCountryRejected(loc)) {
>                        return false;
>                    }
>                } else {
>                    LOGGER.info("Could not determine geolocation for [{}]", clientIp);
>                }
>            }
>            return true;
>        }
>    ```
>
>    配置对象为 AdaptiveAuthenticationProperties
>
>    配置   cas.authn.adaptive.reject.ip.addresses="”(要限制的ip) 基本就这个有用。
>
> 4. 执行登录逻辑，根据返回的不同进入不同的flow节点，登录成功返回success则进入 createTicketGrantingTicket，进入创建TGT的逻辑。
>
>    ```XML
>    <action-state id="realSubmit">
>            <evaluate expression="authenticationViaFormAction"/>
>            <transition on="warn" to="warn"/>
>            <transition on="success" to="createTicketGrantingTicket"/>
>            <transition on="successWithWarnings" to="showAuthenticationWarningMessages"/>
>            <transition on="authenticationFailure" to="handleAuthenticationFailure"/>
>            <transition on="error" to="initializeLoginForm"/>
>        </action-state>
>    ```
>
>    spring容器内的对象名为 createTicketGrantingTicketAction，
>
>    org.apereo.cas.web.config.CasSupportActionsConfiguration 通过这个配置类注入的。
>
>    ```java
>    @RefreshScope
>        @ConditionalOnMissingBean(name = "createTicketGrantingTicketAction")
>        @Bean
>        public Action createTicketGrantingTicketAction() {
>            return new CreateTicketGrantingTicketAction(centralAuthenticationService,
>                authenticationSystemSupport, ticketRegistrySupport);
>        }
>    ```
>
> 5. 创建TGT过程
>
>    ```java
>    @Override
>        public Event doExecute(final RequestContext context) {
>            //获取webflow域中保存的服务信息，里面包含了，需要跳转回的请求地址。
>            final Service service = WebUtils.getService(context);
>            //获取webflow域中保存的注册的服务信息，注册的服务默认是读取json文件获取的，也可以			根据官网文档来进行变更。
>            final RegisteredService registeredService = WebUtils.getRegisteredService(context);
>            //获取登录结果信息构建对象。里面有封装有登录用户信息
>            final AuthenticationResultBuilder authenticationResultBuilder = WebUtils.getAuthenticationResultBuilder(context);
>    		//通过登录结果信息构建对象生产身份认证结果，认证结果中包含登录用户返回值信息，客户端的调用地址。
>            final AuthenticationResult authenticationResult = this.authenticationSystemSupport.finalizeAllAuthenticationTransactions(authenticationResultBuilder, service);
>           //从结果中取出用户信息对象。
>            final Authentication authentication = buildFinalAuthentication(authenticationResult);
>            //从request域，及webflow域中尝试获取tgt串
>            final String ticketGrantingTicket = WebUtils.getTicketGrantingTicketId(context);
>            //获取TGT对象
>            final TicketGrantingTicket tgt = createOrUpdateTicketGrantingTicket(authenticationResult, authentication, ticketGrantingTicket);
>    	//下面就是将获取到的TGT放入各种域中，并返回success,进入对应节点。
>            if (registeredService != null && registeredService.getAccessStrategy() != null) {
>                WebUtils.putUnauthorizedRedirectUrlIntoFlowScope(context, registeredService.getAccessStrategy().getUnauthorizedRedirectUrl());
>            }
>            WebUtils.putTicketGrantingTicketInScopes(context, tgt);
>            WebUtils.putAuthenticationResult(authenticationResult, context);
>            WebUtils.putAuthentication(tgt.getAuthentication(), context);
>    
>            LOGGER.debug("Calculating authentication warning messages...");
>            final Collection<MessageDescriptor> warnings = calculateAuthenticationWarningMessages(tgt, context.getMessageContext());
>            if (!warnings.isEmpty()) {
>                final LocalAttributeMap attributes = new LocalAttributeMap(CasWebflowConstants.ATTRIBUTE_ID_AUTHENTICATION_WARNINGS, warnings);
>                return new EventFactorySupport().event(this, CasWebflowConstants.TRANSITION_ID_SUCCESS_WITH_WARNINGS, attributes);
>            }
>            return success();
>        }
>    ```
>
>    获取TGT对象详解。
>
>    ​	createOrUpdateTicketGrantingTicket(authenticationResult, authentication, ticketGrantingTicket);
>
>    ```java
>     protected TicketGrantingTicket createOrUpdateTicketGrantingTicket(final AuthenticationResult authenticationResult,
>                                                                          final Authentication authentication, final String ticketGrantingTicket) {
>            final TicketGrantingTicket tgt;
>         //判断从域中获取到的TGT串是否可用，是否需要重新创建。
>            if (shouldIssueTicketGrantingTicket(authentication, ticketGrantingTicket)) {
>                LOGGER.debug("Attempting to issue a new ticket-granting ticket...");
>                tgt = this.centralAuthenticationService.createTicketGrantingTicket(authenticationResult);
>            } else {
>                //如果可用，则更新一下TGT中的用户信息。
>                LOGGER.debug("Updating the existing ticket-granting ticket [{}]...", ticketGrantingTicket);
>                tgt = this.centralAuthenticationService.getTicket(ticketGrantingTicket, TicketGrantingTicket.class);
>                tgt.getAuthentication().update(authentication);
>                this.centralAuthenticationService.updateTicket(tgt);
>            }
>            return tgt;
>        }
>    ```
>
> 6. 创建完TGT ， org.apereo.cas.web.flow.login.SendTicketGrantingTicketAction进入这个节点
>
>    org.apereo.cas.web.support.CookieRetrievingCookieGenerator.addCookie(org.springframework.webflow.execution.RequestContext, java.lang.String) 在节点中会执行添加cookie方法。
>
>    在方法中会根据配置将TGT@ip@userAgent进行连接，然后再进行加密，通过如下代码
>
>    ```java
>    final String theCookieValue = this.casCookieValueManager.buildCookieValue(cookieValue, request);
>    ```
>
>    把生成的值放入cookie中，命名为TGC。
>
> 7. 在进入 org.apereo.cas.web.flow.GenerateServiceTicketAction,通过如下代码生成ST,放入域中
>
>    ```java
>    final ServiceTicket serviceTicketId = this.centralAuthenticationService.grantServiceTicket(ticketGrantingTicket, service, authenticationResult);
>    ```
>
> 8. 后续就是重定向会客户端服务，并将ST当成参数来进行访问，客户端获取到ST后，使用ST到服务端来认证，认证成功直接可以访问资源了。