# FAQ

* _安装Istio后如何打开/关闭mTLS加密？_

	开启/关闭mTLS的最直接方式是完整卸载并重新安装Istio。

	如果你是高级用户并了解其中的风险，也可以这样操作：

    ```bash
    kubectl edit configmap -n istio-system istio
    ```

	通过注释掉或取消注释`authPolicy: MUTUAL_TLS`来切换mTLS，然后执行

    ```bash
    kubectl delete pods -n istio-system -l istio=pilot
    ```

	来重启Pilot，数秒钟后（取决于设置的`*RefreshDelay`）Envoy代理将会从Pilot感知到变化；期间服务可能不可用。

	我们正开发一个更平滑的解决方案。

* _使用Istio Auth的服务是否可以不经过Istio允许而与另一个服务通信？_

	目前不支持，但将来很快会支持。

* _是否支持同一个集群中的部分服务使用Istio Auth，而其他服务不使用？_

	目前不支持，但将来很快会支持。

* _如何在Istio Auth打开的情况下，使用Kubernetes的存活和状态相关的服务健康检查？_

	如果Istio Auth打开，Kubelet中的HTTP和TCP健康检查将无法工作，因为它们没有Istio Auth发行的证书。解决方案就是，使用[liveness command](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/#define-a-liveness-command)做健康检查，比如可以在服务pod中安装curl，并在pod中使用curl访问自身。Istio团队正在积极地开发解决方案。

	readinessProbe的示例：

    ```bash
    livenessProbe:
    exec:
      command:
      - curl
      - -f
      - http://localhost:8080/healthz # Replace port and URI by your actual health check
    initialDelaySeconds: 10
    periodSeconds: 5
    ```

* _Auth打开时可以访问Kubernetes API服务器吗？_

	Kubernetes API服务器不支持交互TLS认证。因此，当Istio的mTLS认证打开时，Istio sidecar pod现阶段并不能与Kubernetes API服务器通信。

