---
reviewers:
- femrtnz
- jcjesus
title: Imagens
content_type: concept
weight: 10
---

<!-- overview -->

Uma imagem de contêiner representa dados binários que encapsulam uma aplicação e todas as suas dependências de software. As imagens de contêiner são pacotes de software executáveis que podem ser executados de forma autônoma e que fazem suposições muito bem definidas sobre seu agente de execução do ambiente.

Normalmente, você cria uma imagem de contêiner da sua aplicação e a envia para um registro antes de fazer referência a ela em um {{< glossary_tooltip text="Pod" term_id="pod" >}}

Esta página fornece um resumo sobre o conceito de imagem de contêiner.  

<!-- A container image represents binary data that encapsulates an application and all its software dependencies. Container images are executable software bundles that can run standalone and that make very well defined assumptions about their runtime environment.

You typically create a container image of your application and push it to a registry before referring to it in a Pod

This page provides an outline of the container image concept. -->

<!-- body -->

## Nomes das imagens

As imagens de contêiner geralmente recebem um nome como `pause`, `exemplo/meuconteiner`, ou `kube-apiserver`.
As imagens também podem incluir um hostname de algum registro; por exemplo: `exemplo.registro.ficticio/nomeimagem`,
e um possível número de porta; por exemplo: `exemplo.registro.ficticio:10443/nomeimagem`.

<!-- Container images are usually given a name such as `pause`, `example/mycontainer`, or `kube-apiserver`.
Images can also include a registry hostname; for example: `fictional.registry.example/imagename`,
and possible a port number as well; for example: `fictional.registry.example:10443/imagename`. -->

Se você não especificar um hostname de registro, o Kubernetes presumirá que você se refere ao registro público do Docker.
<!-- If you don't specify a registry hostname, Kubernetes assumes that you mean the Docker public registry. -->

Após a parte do nome da imagem, você pode adicionar uma _tag_ (como também usar com comandos como
como `docker` e` podman`).
As tags permitem identificar diferentes versões da mesma série de imagens.
<!-- After the image name part you can add a _tag_ (as also using with commands such
as `docker` and `podman`).
Tags let you identify different versions of the same series of images. -->

Tags de imagem consistem em letras maiúsculas e minúsculas, dígitos, sublinhados (`_`),
pontos (`.`) e travessões (` -`).
Existem regras adicionais sobre onde você pode colocar o separador
caracteres (`_`,` -` e `.`) dentro de uma tag de imagem.
Se você não especificar uma tag, o Kubernetes presumirá que você se refere à tag `latest` (mais recente).
<!-- Image tags consist of lowercase and uppercase letters, digits, underscores (`_`),
periods (`.`), and dashes (`-`).  
There are additional rules about where you can place the separator
characters (`_`, `-`, and `.`) inside an image tag.  
If you don't specify a tag, Kubernetes assumes you mean the tag `latest`. -->

{{< caution >}}
Você deve evitar usar a tag `latest` quando estiver realizando o deploy de contêineres em produção,
pois é mais difícil rastrear qual versão da imagem está sendo executada, além de tornar mais difícil o processo de reversão para uma versão funcional.

Em vez disso, especifique uma tag significativa, como `v1.42.0`.
{{< /caution >}}
<!-- {{< caution >}}
You should avoid using the `latest` tag when deploying containers in production,
as it is harder to track which version of the image is running and more difficult
to roll back to a working version.

Instead, specify a meaningful tag such as `v1.42.0`.
{{< /caution >}} -->

## Atualizando imagens
<!-- ## Updating images -->

A política de pull padrão é `IfNotPresent` que faz com que o
{{<glossary_tooltip text = "kubelet" term_id = "kubelet">}} ignore 
o processo de *pull* da imagem, caso a mesma já exista. Se você prefere sempre forçar o processo de *pull*, 
você pode fazer como exemplo:
<!-- The default pull policy is `IfNotPresent` which causes the
{{< glossary_tooltip text="kubelet" term_id="kubelet" >}} to skip
pulling an image if it already exists. If you would like to always force a pull,
you can do one of the following: -->

- defina a `imagePullPolicy` do contêiner para` Always`.
- omita `imagePullPolicy` e use`: latest` como a tag para a imagem a ser usada.
- omita o `imagePullPolicy` e a tag da imagem a ser usada.
- habilite o [AlwaysPullImages](/docs/reference/access-authn-authz/admission-controllers/#alwayspullimages) controlador de admissão.
<!-- - set the `imagePullPolicy` of the container to `Always`.
- omit the `imagePullPolicy` and use `:latest` as the tag for the image to use.
- omit the `imagePullPolicy` and the tag for the image to use.
- enable the [AlwaysPullImages](/docs/reference/access-authn-authz/admission-controllers/#alwayspullimages) admission controller. -->

Quando `imagePullPolicy` é definido sem um valor específico, ele também é definido como` Always`.
<!-- When `imagePullPolicy` is defined without a specific value, it is also set to `Always`. -->

## Multiarquitetura de imagens com índice de imagens
<!-- ## Multi-architecture images with image indexes -->

Além de fornecer o binário das imagens, um registro de contêiner também pode servir um [índice de imagem do contêiner](https://github.com/opencontainers/image-spec/blob/master/image-index.md). Um índice de imagem pode apontar para multiplos [manifestos da imagem](https://github.com/opencontainers/image-spec/blob/master/manifest.md) para versões específicas de arquitetura de um contêiner. A ideia é que você possa ter um nome para uma imagem (por exemplo: `pause`, `exemple/meuconteiner`, `kube-apiserver`) e permitir que diferentes sistemas busquem o binário da imagem correta para a arquitetura de máquina que estão usando.
<!-- As well as providing binary images, a container registry can also serve a [container image index](https://github.com/opencontainers/image-spec/blob/master/image-index.md). An image index can point to multiple [image manifests](https://github.com/opencontainers/image-spec/blob/master/manifest.md) for architecture-specific versions of a container. The idea is that you can have a name for an image (for example: `pause`, `example/mycontainer`, `kube-apiserver`) and allow different systems to fetch the right binary image for the machine architecture they are using. -->

<!-- Tipicamente o próprio Kubernetes nomeia as imagens dos contêineres com o sufixo `-$(ARCH)`. Para retro-compatibilidade, gere as imagens mais antigas com sufixos. A ideia é gerar a imagem `pause` que contém o manifesto para todas as arquiteturas e ` pause-amd64` que é retro-compativel com as configurações anteriores ou arquivos YAML que podem ter codificado as imagens com sufixos.

O próprio Kubernetes normalmente nomeia as imagens de contêiner com o sufixo `- $ (ARCH)`. Para retrocompatibilidade, gere as imagens mais antigas com sufixos. A ideia é gerar a imagem `pause` que tem o manifesto para todas as arquiteturas e `pause-amd64` que é retrocompatível com as configurações anteriores ou arquivos YAML que podem ter codificado as imagens com sufixos. -->
Kubernetes itself typically names container images with a suffix `-$(ARCH)`. For backward compatibility, please generate the older images with suffixes. The idea is to generate say `pause` image which has the manifest for all the arch(es) and say `pause-amd64` which is backwards compatible for older configurations or YAML files which may have hard coded the images with suffixes.

## Usando um registro privado
<!-- ## Using a private registry -->

Os registros privados podem exigir chaves para acessar as imagens deles.
As credenciais podem ser fornecidas de várias maneiras:
<!-- Private registries may require keys to read images from them.  
Credentials can be provided in several ways: -->
  - Configurando nós para autenticação em um registro privado
     - todos os pods podem ler qualquer registro privado configurado
     - requer configuração de nó pelo administrador do cluster
   - Imagens pré-obtidas
     - todos os pods podem usar qualquer imagem armazenada em cache em um nó
     - requer acesso root a todos os nós para configurar
   - Especificando ImagePullSecrets em um Pod
     - apenas pods que fornecem chaves próprias podem acessar o registro privado
   - Extensões locais ou específicas do fornecedor
     - se estiver usando uma configuração de nó personalizado, você (ou seu provedor de nuvem) pode implementar seu mecanismo para autenticar o nó ao registro do contêiner.
  <!-- - Configuring Nodes to Authenticate to a Private Registry
    - all pods can read any configured private registries
    - requires node configuration by cluster administrator
  - Pre-pulled Images
    - all pods can use any images cached on a node
    - requires root access to all nodes to setup
  - Specifying ImagePullSecrets on a Pod
    - only pods which provide own keys can access the private registry
  - Vendor-specific or local extensions
    - if you're using a custom node configuration, you (or your cloud
      provider) can implement your mechanism for authenticating the node
      to the container registry. -->

Essas opções são explicadas com mais detalhes abaixo.
<!-- These options are explained in more detail below. -->

### Configurando nós para autenticação em um registro privado
<!-- ### Configuring nodes to authenticate to a private registry -->

Se você executar o Docker em seus nós, poderá configurar o contêiner do Docker
runtime para autenticação em um registro de contêiner privado.

Essa abordagem é adequada se você puder controlar a configuração do nó.
<!-- If you run Docker on your nodes, you can configure the Docker container
runtime to authenticate to a private container registry.

This approach is suitable if you can control node configuration. -->

{{< note >}}
O Kubernetes padrão é compatível apenas com as seções `auths` e` HttpHeaders` na configuração do Docker.
Auxiliares de credencial do Docker (`credHelpers` ou` credsStore`) não são suportados.
{{< /note >}}
<!-- 
{{< note >}}
Default Kubernetes only supports the `auths` and `HttpHeaders` section in Docker configuration.
Docker credential helpers (`credHelpers` or `credsStore`) are not supported.
{{< /note >}} -->

Docker armazena chaves de registros privados no arquivo `$HOME/.dockercfg` ou `$HOME/.docker/config.json`. Se você colocar o mesmo arquivo na lista de caminhos de pesquisa abaixo, o kubelet o usa como provedor de credenciais ao obter imagens.
<!-- Docker stores keys for private registries in the `$HOME/.dockercfg` or `$HOME/.docker/config.json` file.  If you put the same file
in the search paths list below, kubelet uses it as the credential provider when pulling images. -->

* `{--root-dir:-/var/lib/kubelet}/config.json`
* `{cwd of kubelet}/config.json`
* `${HOME}/.docker/config.json`
* `/.docker/config.json`
* `{--root-dir:-/var/lib/kubelet}/.dockercfg`
* `{cwd of kubelet}/.dockercfg`
* `${HOME}/.dockercfg`
* `/.dockercfg`

{{< note >}}
Você talvez tenha que definir `HOME = / root` explicitamente no ambiente do processo kubelet.
{{< /note >}}

<!-- {{< note >}}
You may have to set `HOME=/root` explicitly in the environment of the kubelet process.
{{< /note >}} -->

Aqui estão as etapas recomendadas para configurar seus nós para usar um registro privado. Neste
exemplo, execute-os em seu desktop/laptop:
<!-- Here are the recommended steps to configuring your nodes to use a private registry.  In this
example, run these on your desktop/laptop: -->

  1. Execute `docker login [servidor]` para cada conjunto de credenciais que deseja usar. Isso atualiza `$HOME/.docker/config.json` em seu PC.
  1. Visualize `$HOME/.docker/config.json` em um editor para garantir que contém apenas as credenciais que você deseja usar.
  1. Obtenha uma lista de seus nós; por exemplo:
      - se você quiser os nomes: `nodes=$( kubectl get nodes -o jsonpath='{range.items[*].metadata}{.name} {end}' )`
      - se você deseja obter os endereços IP: `nodes=$( kubectl get nodes -o jsonpath='{range .items[*].status.addresses[?(@.type=="ExternalIP")]}{.address} {end}' )`
  1. Copie seu `.docker/config.json` local para uma das listas de caminhos de busca acima.
      - por exemplo, para testar isso: `for n in $nodes; do scp ~/.docker/config.json root@"$n":/var/lib/kubelet/config.json; done`
   <!-- 1. Run `docker login [server]` for each set of credentials you want to use.  This updates `$HOME/.docker/config.json` on your PC.
   1. View `$HOME/.docker/config.json` in an editor to ensure it contains just the credentials you want to use.
   1. Get a list of your nodes; for example:
      - if you want the names: `nodes=$( kubectl get nodes -o jsonpath='{range.items[*].metadata}{.name} {end}' )`
      - if you want to get the IP addresses: `nodes=$( kubectl get nodes -o jsonpath='{range .items[*].status.addresses[?(@.type=="ExternalIP")]}{.address} {end}' )`
   1. Copy your local `.docker/config.json` to one of the search paths list above.
      - for example, to test this out: `for n in $nodes; do scp ~/.docker/config.json root@"$n":/var/lib/kubelet/config.json; done` -->

{{< note >}}
Para clusters de produção, use uma ferramenta de gerenciamento de configuração para que você possa aplicar esta
configuração para todos os nós onde você precisa.
{{< /note >}}
<!-- 
{{< note >}}
For production clusters, use a configuration management tool so that you can apply this
setting to all the nodes where you need it.
{{< /note >}} -->

Verifique se está funcionando criando um pod que usa uma imagem privada; por exemplo:
<!-- Verify by creating a Pod that uses a private image; for example: -->

```shell
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: private-image-test-1
spec:
  containers:
    - name: uses-private-image
      image: $PRIVATE_IMAGE_NAME
      imagePullPolicy: Always
      command: [ "echo", "SUCCESS" ]
EOF
```
```
pod/private-image-test-1 created
```

Se tudo estiver funcionando, então, após algum tempo, você pode executar:
<!-- If everything is working, then, after a few moments, you can run: -->

```shell
kubectl logs private-image-test-1
```
e veja o resultado do comando:
<!-- and see that the command outputs: -->
```
SUCCESS
```

Se você suspeitar que o comando falhou, você pode executar:
<!-- If you suspect that the command failed, you can run: -->
```shell
kubectl describe pods/private-image-test-1 | grep 'Failed'
```
Em caso de falha, a saída é semelhante a:
<!-- In case of failure, the output is similar to: -->
```
  Fri, 26 Jun 2015 15:36:13 -0700    Fri, 26 Jun 2015 15:39:13 -0700    19    {kubelet node-i2hq}    spec.containers{uses-private-image}    failed        Failed to pull image "user/privaterepo:v1": Error: image user/privaterepo:v1 not found
```


Você deve garantir que todos os nós no cluster tenham o mesmo `.docker/config.json`. Caso contrário, os pods serão executados em
alguns nós e não funcionam em outros. Por exemplo, se você usar o escalonamento automático de nós, cada instância
o modelo precisa incluir o `.docker / config.json` ou montar uma unidade que o contenha.
Você deve garantir que todos os nós no cluster tenham o mesmo `.docker/config.json`. Caso contrário, os pods serão executados com sucesso em alguns nós e falharão em outros. Por exemplo, se você usar o escalonamento automático de nós, cada modelo de instância precisa incluir o `.docker/config.json` ou montar um drive que o contenha.
<!-- You must ensure all nodes in the cluster have the same `.docker/config.json`.  Otherwise, pods will run on
some nodes and fail to run on others.  For example, if you use node autoscaling, then each instance
template needs to include the `.docker/config.json` or mount a drive that contains it. -->

Todos os pods terão acesso de leitura às imagens em qualquer registro privado, uma vez privado
as chaves de registro são adicionadas ao `.docker/config.json`.
<!-- All pods will have read access to images in any private registry once private
registry keys are added to the `.docker/config.json`. -->

### Imagens pré-obtidas
<!-- ### Pre-pulled images -->

{{< note >}}
This approach is suitable if you can control node configuration.  It
will not work reliably if your cloud provider manages nodes and replaces
them automatically.
{{< /note >}}

By default, the kubelet tries to pull each image from the specified registry.
However, if the `imagePullPolicy` property of the container is set to `IfNotPresent` or `Never`,
then a local image is used (preferentially or exclusively, respectively).

If you want to rely on pre-pulled images as a substitute for registry authentication,
you must ensure all nodes in the cluster have the same pre-pulled images.

This can be used to preload certain images for speed or as an alternative to authenticating to a private registry.

All pods will have read access to any pre-pulled images.

### Specifying imagePullSecrets on a Pod

{{< note >}}
This is the recommended approach to run containers based on images
in private registries.
{{< /note >}}

Kubernetes supports specifying container image registry keys on a Pod.

#### Creating a Secret with a Docker config

Run the following command, substituting the appropriate uppercase values:

```shell
kubectl create secret docker-registry <name> --docker-server=DOCKER_REGISTRY_SERVER --docker-username=DOCKER_USER --docker-password=DOCKER_PASSWORD --docker-email=DOCKER_EMAIL
```

If you already have a Docker credentials file then, rather than using the above
command, you can import the credentials file as a Kubernetes
{{< glossary_tooltip text="Secrets" term_id="secret" >}}.  
[Create a Secret based on existing Docker credentials](/docs/tasks/configure-pod-container/pull-image-private-registry/#registry-secret-existing-credentials) explains how to set this up.

This is particularly useful if you are using multiple private container
registries, as `kubectl create secret docker-registry` creates a Secret that
only works with a single private registry.

{{< note >}}
Pods can only reference image pull secrets in their own namespace,
so this process needs to be done one time per namespace.
{{< /note >}}

#### Referring to an imagePullSecrets on a Pod

Now, you can create pods which reference that secret by adding an `imagePullSecrets`
section to a Pod definition.

For example:

```shell
cat <<EOF > pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: foo
  namespace: awesomeapps
spec:
  containers:
    - name: foo
      image: janedoe/awesomeapp:v1
  imagePullSecrets:
    - name: myregistrykey
EOF

cat <<EOF >> ./kustomization.yaml
resources:
- pod.yaml
EOF
```

This needs to be done for each pod that is using a private registry.

However, setting of this field can be automated by setting the imagePullSecrets
in a [ServiceAccount](/docs/tasks/configure-pod-container/configure-service-account/) resource.

Check [Add ImagePullSecrets to a Service Account](/docs/tasks/configure-pod-container/configure-service-account/#add-imagepullsecrets-to-a-service-account) for detailed instructions.

You can use this in conjunction with a per-node `.docker/config.json`.  The credentials
will be merged.

## Use cases

There are a number of solutions for configuring private registries.  Here are some
common use cases and suggested solutions.

1. Cluster running only non-proprietary (e.g. open-source) images.  No need to hide images.
   - Use public images on the Docker hub.
     - No configuration required.
     - Some cloud providers automatically cache or mirror public images, which improves availability and reduces the time to pull images.
1. Cluster running some proprietary images which should be hidden to those outside the company, but
   visible to all cluster users.
   - Use a hosted private [Docker registry](https://docs.docker.com/registry/).
     - It may be hosted on the [Docker Hub](https://hub.docker.com/signup), or elsewhere.
     - Manually configure .docker/config.json on each node as described above.
   - Or, run an internal private registry behind your firewall with open read access.
     - No Kubernetes configuration is required.
   - Use a hosted container image registry service that controls image access
     - It will work better with cluster autoscaling than manual node configuration.
   - Or, on a cluster where changing the node configuration is inconvenient, use `imagePullSecrets`.
1. Cluster with proprietary images, a few of which require stricter access control.
   - Ensure [AlwaysPullImages admission controller](/docs/reference/access-authn-authz/admission-controllers/#alwayspullimages) is active. Otherwise, all Pods potentially have access to all images.
   - Move sensitive data into a "Secret" resource, instead of packaging it in an image.
1. A multi-tenant cluster where each tenant needs own private registry.
   - Ensure [AlwaysPullImages admission controller](/docs/reference/access-authn-authz/admission-controllers/#alwayspullimages) is active. Otherwise, all Pods of all tenants potentially have access to all images.
   - Run a private registry with authorization required.
   - Generate registry credential for each tenant, put into secret, and populate secret to each tenant namespace.
   - The tenant adds that secret to imagePullSecrets of each namespace.


If you need access to multiple registries, you can create one secret for each registry.
Kubelet will merge any `imagePullSecrets` into a single virtual `.docker/config.json`

## {{% heading "whatsnext" %}}

* Read the [OCI Image Manifest Specification](https://github.com/opencontainers/image-spec/blob/master/manifest.md)
