

# 基本概念

随着上下文不同，“权限”所指代的东西也是不同的，本文讨论的是在计算机领域下应用的“授权”（authorization），与之关联的几个概念有：

- “验证”（authentication）：简称 AuthN，是指确认某人或某事的身份的过程。在计算机安全领域，这通常涉及到确认一个用户或系统是否真的是它声称的那个实体。验证可以通过多种方式进行，包括密码、生物识别技术、智能卡或数字证书等。验证是访问控制过程的第一步，只有在成功验证身份后，系统才会进一步考虑授予访问权限。其涉及到的技术协议和框架有：OAuth, OpenID 及 SAML 等。
- “授权”（authorization）：简称 AuthZ，在用户的身份得到验证后，系统需要决定该用户能够访问哪些资源，以及可以执行哪些操作。授权是一个定义和管理访问权限的过程，它决定了验证过的用户可以查看、使用、修改或删除哪些数据或资源，旨在确保只有得到适当权限的用户才能访问特定的资源或数据。其涉及到的授权模型有基于角色的访问控制（RBAC）、基于属性的访问控制（ABAC） 等。
- “权限控制”（access control）：是一种更广泛的概念，涵盖了验证和授权的过程，是指限制对系统和数据的访问和使用的各种方法和技术的总称。

# 使用场景

下面列举一些研发场景中会遇到的权限场景。

## 文件系统权限
![](attachments/Pasted%20image%2020251005153118.png)

图中的 `drwx------` 即表示项目的权限，其中第一个字符表示文件类型（`-`表示普通文件，`d`表示目录），接下来的三组三个字符分别代表所有者、组和其他用户的读、写和执行权限。但我们看到有的字符串后面还跟着 `@` 和 `+` 符号，其中 `@` 符号表示当前文件或目录有扩展属性，此处不讨论；而 `+` 符号表示存在访问控制列表（Access Control List, ACL）。ACL 是一种用于定义额外权限的机制，允许对文件或目录进行更细粒度的访问控制，这意味着除了标准的文件权限（如读、写、执行）之外，还有额外的权限设置，这些设置可以针对不同的用户或组进行详细配置。使用 `ls -le` 操作后，我们可以看到这些访问控制列表：
![](attachments/Pasted%20image%2020251005153155.png)

更具体的说明可以通过 `man chmod` 查看，这里我们有 2 点观察：

- Unix 的权限粒度过粗，只支持读、写和执行 3 种不同操作的权限管控，其他操作系统选择添加额外的 ACL 机制来打补丁。
- macOS ACL 的规则格式为： `（用户，允许 / 拒绝，操作）` 的三元组。
![](attachments/Pasted%20image%2020251005153217.png)截图来自 macOS 14.2 manual

## 数据库权限

数据库系统提供一系列的权限或权限组合，使数据库管理员可以为用户或用户组分配、审计权限。

下面的一系列截图是一组授权和查看权限的例子：
![](attachments/Pasted%20image%2020251005153234.png)授权：为用户名为 ‘admin’ 的用户授予 ‘lark.message’ 表的 SELECT 权限。
![](attachments/Pasted%20image%2020251005153246.png)查看用户的全局权限：‘admin’用户没有任何全局权限。
![](attachments/Pasted%20image%2020251005153256.png)查看一个用户对一个数据表的权限：‘admin’用户对‘lark.message‘表有 SELECT 权限。

从上面的例子中我们也有如下观察：

1. MySQL 的权限控制是有多个层级的，一个用户有全局范围的权限，也有针对具体表范围的权限。
2. MySQL 的权限控制也是`（用户，允许 / 拒绝，操作）` 的三元组。

# 访问控制模型

业界常用的访问控制模型有多种，如：自主访问控制（Discretionary Access Control, DAC）、强制访问控制（Mandatory Access Control, MAC）、基于角色的访问控制（Role-Based Access Control, RBAC）、基于属性的访问控制（Attribute-Based Access Control, ABAC）、基于策略的访问控制（Policy-Based Access Control, PBAC）。

> [!note] 💡 请注意：虽然相关的访问控制模型算是一种共识，但目前业界并没有统一的标准去判定一个现实中的方案属于哪一个访问控制模型，因此下面的说明主要是起到概念说明的作用，而不是做明确的定义和区分。

## DAC

DAC 是一种最基本的访问控制模型，允许资源的所有者对资源的访问进行控制。在 DAC 模型中，用户可以根据自己的判断将访问权限分配给其他用户。

上面举的文件系统权限和数据库权限的例子，其实都是 DAC 系统，直接指定了“谁可以做什么操作”，如文件系统的 `User 1 allow read` ，数据库的 `GRANT SELECT ON lark.message TO 'admin'@'%';`。

## RBAC

RBAC 是一种以用户的角色为基础进行访问控制的模型，它允许系统管理员根据组织的职能和职责将用户分配到不同的角色中，每个角色都有一组预定义的访问权限。RBAC 简化了权限管理，因为管理员只需管理角色与权限的关系，而不是每个用户的权限。

下面的这篇文章介绍了如何在 SAP 中配置 RBAC：

[SAP Commissions - Implementing Authorization With User Roles (RBAC)](https://community.sap.com/t5/human-capital-management-blogs-by-sap/sap-commissions-implementing-authorization-with-user-roles-rbac/ba-p/13554527)

通常在 RBAC 中，我们会有用户组的概念，用户组把用户和具体的授权绑定在一起。
![](attachments/Pasted%20image%2020251005153322.png)

RBAC 有简化管理、灵活可配置的优点，但实践中也会表现出如下缺点：

1. **角色膨胀**：在复杂的组织中，可能需要创建大量的角色来覆盖所有的访问需求，这可能导致角色管理变得复杂和难以维护。
2. **初始设置复杂**：正确实施RBAC系统可能需要大量的前期规划和配置，特别是在将现有权限迁移到RBAC模型的过程中。
3. **灵活性限制**：虽然RBAC提高了权限管理的效率，但在某些情况下，它可能限制了对个别用户特定需求的灵活性。
4. **性能问题**：在拥有大量用户、角色和权限规则的大型系统中，权限检查可能会导致性能问题。
5. **管理挑战**：随着组织的发展，管理和更新角色及其权限可能变得复杂，尤其是在缺乏自动化工具支持的情况下。

## ABAC

ABAC 是一种更为灵活和动态的访问控制模型，它根据访问请求者的属性（如年龄、职位等）、资源的属性和环境条件（如时间）来决定访问权限。ABAC 提供了更细粒度的访问控制，支持更复杂的安全策略，适用于需要高度定制化访问控制的场景。

ABAC 相对于 RBAC 的优点，主要在于 RBAC 的授权基于用户组，一个组内的成员权限互通，用一个医疗系统举例，如果我们希望一个护士只能看见 ta 负责的病人病历，除非给每个护士创建一个用户组，不然无法用 RBAC 实现。而 RBAC 只需要 `病历负责护士 == 当前用户` 这样一个规则就可以实现。

一种现实中的实践是，用 RBAC 分发规则组，及一个用户需要哪些授权规则，到具体的授权规则上，则是 ABAC，即考虑用户和实体的属性来决定是否授权。

## 现实案例

[https://www.youtube.com/watch?v=ZUmzELJ2UcM&list=PLnobS_RgN7JZxK1wjUvQ84jMFqRZoJXbD&index=1](https://www.youtube.com/watch?v=ZUmzELJ2UcM&list=PLnobS_RgN7JZxK1wjUvQ84jMFqRZoJXbD&index=1)

![](attachments/Pasted%20image%2020251005153344.png)

视频中展示了 4 中访问控制：

1. IP、登陆时间决定能否访问组织。
2. 用户权限组决定能否访问对象。
3. 角色、汇报线决定能否访问具体记录。
4. 用户权限组决定能否访问一个记录上的具体字段。

可以看到不同的权限访问控制模型存在于系统的不同层级，这也是权限系统的一个抽象难点。

# 业界解决方案

早期业界的很多解决方案并不会刻意区分认证（AuthN）和授权（AuthZ），在我自己的观察中，RBAC 通常是这类框架首要支持的访问控制模型，因为认证是围绕着用户进行的，那么之后的访问控制围绕着用户展开也在情理之中。

- [Spring Security](https://github.com/spring-projects/spring-security)：没有控制面板、提供了各类 API，但不支持配置文件。
- [Apache Shiro](https://github.com/apache/shiro)：没有控制面板、支持配置文件，但不支持 ABAC。

上面的两个解决方案及其他较早的开源方案对访问控制模型的支持都比较简单，因此也不在本文的讨论范围内，而近年有两个方案对访问控制模型的支持则更加全面，分别是：

- [Casbin](https://github.com/casbin/casbin)：开源，定位是一个 SDK，支持各类访问控制模型，对现存的各类系统都有集成，通过配置定义权限。
- [Zanzibar](https://research.google/pubs/zanzibar-googles-consistent-global-authorization-system/)：闭源，定位是一个鉴权服务，也支持各类访问各类控制模型，控制粒度更细，权限通过接口调用配置。

## Casbin

下面给一个 Casbin 创建 RBAC 的例子。

model 文件：

```yaml
[request_definition]
r = sub, act, obj

[policy_definition]
p = sub, act, obj

[role_definition]
g = _, _
g2 = _, _

[policy_effect]
e = some(where (p.eft == allow))

[matchers]
m = r.sub == p.sub && g(p.act, r.act) && g2(p.obj, r.obj)
```

policy 文件：

```yaml
p, alice, sub-reader, sub1
p, bob, rg-owner, rg2

// subscription role to subscription action mapping
g, sub-reader, sub-read
g, sub-owner, sub-read
g, sub-owner, sub-write

// resourceGroup role to resourceGroup action mapping
g, rg-reader, rg-read
g, rg-owner, rg-read
g, rg-owner, rg-write

// subscription role to resourceGroup role mapping
g, sub-reader, rg-reader
g, sub-owner, rg-owner

// subscription resource to resourceGroup resource mapping
g2, sub1, rg1
g2, sub2, rg2
```

请求： `alice, rg-read, rg1` ，返回： `true`。

其推理过程为，先匹配用户，alice 对应的配置是： `alice, sub-reader, sub1`
![](attachments/Pasted%20image%2020251005153405.png)
然后匹配操作，请求的操作是 `rg-read`，配置的 alice 操作角色是 `sub-reader`，配置 `g, sub-reader, rg-reader` 和 `g, rg-reader, rg-read` 可以从 alice 的 `sub-reader` 角色推理出其可以执行 `rg-read` 操作。
![](attachments/Pasted%20image%2020251005153418.png)
最后匹配操作对象，请求的对象是 `rg1` ，配置的 alice 对象角色是 `sub1`，这层关系则可以通过 `g2, sub1, rg1` 配置推理出。
![](attachments/Pasted%20image%2020251005153431.png)
## Zanzibar

关于 Zanzibar 的具体介绍本文略过，下面的网址给出了 Zanzibar 的具体介绍。

[Zanzibar: A Global Authorization System - Presented by Auth0](https://zanzibar.academy/)

值得一提的是 Zanzibar 作为一个独立服务，与 Casbin 作为 SDK 的不同点，独立服务有自己的存储来记录关系数据，也需要自己进行缓存、数据更新冲突等问题，Zanzibar 相比于 Casbin 肯定是一个更成熟的方案，但也意味着更多的运维成本，以及和 SDK 相比，对现有系统可能会有更多的兼容性问题。

# 抽象的难点

权限系统没有像TCP/IP这样的通用标准，主要是因为权限管理的需求和实现方式在不同的应用和环境中差异很大。TCP/IP协议是网络通信的基础，为不同类型的网络提供了一个统一的通信标准，其设计目的是为了实现网络间的互连互通。而权限系统则涉及到更多层面的因素，包括但不限于业务逻辑、组织结构、安全要求等，这些因素在不同的应用场景中会有很大的不同。下面列出了几个为什么权限系统难以有通用标准的原因：

1. **多样性的业务需求**：不同类型的应用有着不同的业务模型和安全需求。例如，金融行业的权限管理系统需要考虑到复杂的合规性和审计要求，而社交媒体平台的权限系统可能更关注用户隐私和内容管理。这种多样性使得难以制定一个覆盖所有场景的统一标准。
2. **组织结构差异**：不同组织的结构、政策和管理流程也会影响权限管理的实现。大型企业可能需要一个复杂的角色层次和细粒度的权限控制，而小型团队可能只需要一个简单的权限模型。
3. **技术发展迅速**：IT技术（包括软件开发框架、数据库技术、云服务等）的快速发展意味着新的权限管理方法和工具不断出现，这使得难以维持一个长期有效的统一标准。
4. **安全性和灵活性的平衡**：权限系统需要在安全性和灵活性之间找到平衡。不同的应用场景可能对这两者的重视程度不同，导致采用不同的权限管理策略。
5. **标准化的难度**：虽然有些权限管理的概念（如基于角色的访问控制RBAC）被广泛接受和使用，但将这些概念扩展到一个通用的、覆盖所有可能用例的标准是非常困难的。尝试制定这样的标准可能会导致过于复杂和笨重，难以适应快速变化的技术和业务需求。

因此，虽然在某些方面（如认证和加密技术）可以有一定程度的标准化，权限管理系统的具体实现往往需要根据特定的应用场景和需求来定制。

## 拓展：零信任网络模型

零信任网络模型更偏向于身份认知（即 AuthN）的内容，但是其许多思想与 AuthZ 的设计息息相关，因此本文最后罗列一下零信任网络模型的知识：

零信任（Zero Trust）是一种网络安全模型，其核心原则是“永不信任，始终验证”。这种模型要求对所有试图访问网络资源的用户和设备进行严格的身份验证，无论它们位于网络的内部还是外部。零信任模型的出发点是不再假设网络内部是安全的，而是认为威胁可能来自任何地方，因此需要对每个访问请求进行验证和授权。

零信任模型的核心原则包括：

1. **最小权限原则**：用户和设备仅获得完成其任务所需的最小权限，限制对敏感信息和系统的访问。
2. **持续验证**：系统持续监控和验证用户和设备的信任状态，任何时候都不会假定它们是安全的。
3. **显式验证**：所有访问尝试都必须进行身份验证和授权，无论用户或设备的位置。
4. **使用多因素认证**（MFA）：增加安全层次，通过要求提供两个或更多证明身份的因素来降低密码泄露的风险。
5. **微分段**：将网络划分为小的、受管理的段，以减少潜在攻击者的移动范围，并限制对敏感数据的访问。
6. **基于风险的适应性策略**：根据用户的行为、设备的安全状态、访问时间和位置等因素，动态调整访问控制策略。

## 引用本文

APA：
Yu, J. (2024年2月25日). 权限模型的迷思. Jiachen Yu. https://www.yujiachen.com/the-permission-model-myth/zh/

BibTeX：
```bibtex
@online{yu2024thepermissionmodelmyth,
  author = {Yu, Jiachen},
  title = {权限模型的迷思},
  year = {2024},
  publisher = {Jiachen Yu},
  url = {https://www.yujiachen.com/the-permission-model-myth/zh/},
  urldate = {2026-02-08},
}
```
