# Springboot

当SpringBoot版本在小于等于2.3.0.RELEASE的情况下， alwaysUseFullPath 为默认值false，这会使得其获取ServletPath，所以在路由匹配时会对``%2e``也即``.``进行解码，这可能导致身份验证绕过。而反过来由于高版本将 alwaysUseFullPath 自动配置成了true从而开启全路径，又可能导致一些安全问题。