#商业用例规范

&nbsp;

### 企业间合约

商业合约能够使涉及到两方或多方的协议合同以一种可信赖的方式自动执行。 尽管区块链上的信息被默认为“公开的”，商业合约也会提供隐私控制机制来保证企业间的敏感信息不会泄露给合约参与者以外的人员。


<img src="../images/Canonical-Use-Cases_B2BContract.png" width="900" height="456">

尽管保密协议是商业用例中的重要的一项，然而也有许多情况合约的内容需要对账本上的所有参与者可见。比如创建一个用来招标的账本，按照规定这个账本应被设计为无权限访问。这种合约需要被标准化以便竞标者可以轻松访问，并能有效通过智能合约（又名：链码）来创建电子贸易平台。

#### 角色

*  合约参与者 – 合约当事人

*  第三方参与者 – 第三方利益相关者，来保证合约的信誉.

#### 重要性质

*  多方签名激活合约 - 当合约首次由某一方合约当事人部署到账本上时，合约会处于等待激活状态。若要激活合约，需要所有其他合约当事人方以及（或者）第三方参与者的签名。

*  多方签名执行合约 - 某些合约在执行时也需要一方或多方签名。比如在贸易金融中，只有当收货人或第三方（如快递公司）确认发货之后，付款指令才会被执行。

*  可发现性 - 如果合约的内容是关于商业招标，这个合约必须易于查找。另外，这样的合约也必须内置评估，选择和调节竞价的机制。 

*  合约分块执行 - 合约分块执行可以保证仅有在货款到账的情况下才会进行货物转移（运送vs支付）. 如果交易过程中任意一个步骤出现问题，交易必须终止并返回交易前状态。 

*  合约与链码间通信 - 合约必须能够和部署在同一个账本上的链码进行通信。

*  更长时间有效的合约 - 企业间合约由于执行时间较长，则需要引入计时器来记录用时。

*  可复用合约 - 对常用合约进行标准化以便反复使用。

*  可审计的合同协议 - 任何合约都应设定为可以被第三方审计。

*  合约生命周期管理 - 企业间合约都是专有的，不可能把所有的合约都标准化。因此一个有效的合约管理系统就尤为重要，该系统可以用来提升账本网络的规模。

*  验证权限 – 只有拥有验证权限的节点才可以验证企业间合约的交易。

*  查看权限 – 企业间合约也会包含保密信息，只有被授与查看权限的账户才能查看和质询这些信息。

&nbsp;


### 制造商供应链

对于一个处在供应链终端的装配企业来说，比如汽车制造商，它可以创建一个由它的同行们和供应商共同维护的供应链网络。这样汽车制造商就能更好地管理它的供应商也能更好的应对汽车召回这样的事件（这类事件通常因为供应商提供了不合规格的零件而引发）。fabric的区块链必须提供标准的协议来确保供应链网络上的每个参与者都能写入或追踪任意一辆汽车上的任意部件。

为什么我们把这个合约用例单独列出来呢? 因为尽管所有区块链主要用来存储不可更改的信息，然而有些区块链应用具有实现在不同实体间转移财物的需求。本例就特别指出深度逆向搜索交易的需求，有时甚至需要搜索5到10层历史交易。在供应链中如果许多成品都是由其他部件装配而成，而这种逆向追踪搜索的能力就能找出一个成品所有部件的原产地。

<img src="../images/Canonical-Use-Cases_Manufacturing-Supply-Chain.png" width="900" height="552">

#### 角色

*  终端装配企业 – 供应链终端把零部件装配成最终成品的企业。

*  部件供应商 – 供应零部件的供应商。 供应商也会把下级供应商提供的更零散的部件装配成其他部件或成品，然后提供给终端装配企业。

#### 重要部分

*  货到后付款 - 需要继承链外支付系统，区块链可以保证当部件送达后再执行支付指令。

*  第三方审计 -  所有的部件必须对第三方开发审计验收。比如，监管者可能需要追踪某一部件供应商供应的部件的总数，为了税务方面原因。

*  发货过程不可见 - 发货结算数额必须设为不可见否则供应商可以借此推测出同行的商业活动信息。

*  市场规模不可见 - 结算数额总数不许设为不可见否则供应商可以借此推测出自己所占的市场份额，由此可能引发制定合约条款时非公正竞争。

*  验证权限 – 只有拥有验证权限的节点才可以验证交易（部件发货）。

*  查看权限 – 只有被授与查看权限的账户才能质询运送的部件以及可用部件的结算数额。

&nbsp;


### Asset Depository

Assets such as financial securities must be able to be dematerialized on a blockchain network so that all stakeholders of an asset type will have direct access to that asset, allowing them to initiate trades and acquire information on an asset without going through layers of intermediaries. Trades should be settled in near real time and all stakeholders must be able to access asset information in near real time. A stakeholder should be able to add business rules on any given asset type, as one example of using automation logic to further reduce operating costs.
<img src="../images/Canonical-Use-Cases_Asset-Depository.png" width="900" height="464">

#### 角色

*  Investor – Beneficial and legal owner of an asset.

*  Issuer – Business entity that issued the asset which is now dematerialized on the ledger network.

*  Custodian – Hired by investors to manage their assets, and offer other value-add services on top of the assets being managed.

*  Securities Depository – Depository of dematerialized assets.

#### 重要部分

*  Asset to cash - Integration with off-chain payment systems is necessary so that issuers can make payments to and receive payments from investors.

*  Reference Rate - Some types of assets (such as floating rate notes) may have attributes linked to external data (such as  reference rate), and such information must be fed into the ledger network.

*  Asset Timer - Many types of financial assets have predefined life spans and are required to make periodic payments to their owners, so a timer is required to automate the operation management of these assets.

*  Asset Auditor - Asset transactions must be made auditable to third parties. For example, regulators may want to audit transactions and movements of assets to measure market risks.

*  Obfuscation of account balances - Individual account balances must be obfuscated so that no one can deduce the exact amount that an investor owns.

*  Validation Access – Only nodes with validation rights are allowed to validate transactions that update the balances of an asset type (this could be restricted to CSD and/or the issuer).

*  View access – Only accounts with view access rights are allowed to interrogate the chaincode that defines an asset type. If an asset represents shares of publicly traded companies, then the view access right must be granted to every entity on the network.

&nbsp;


# Extended Use Cases

The following extended use cases examine additional requirements and scenarios.

### One Trade, One Contract

From the time that a trade is captured by the front office until the trade is finally settled, only one contract that specifies the trade will be created and used by all participants. The middle office will enrich the same electronic contract submitted by the front office, and that same contract will then be used by counter parties to confirm and affirm the trade. Finally, securities depository will settle the trade by executing the trading instructions specified on the contract. When dealing with bulk trades, the original contract can be broken down into sub-contracts that are always linked to the original parent contract.

<img src="../images/Canonical-Use-Cases_One-Trade-One-Contract.png" width="900" height="624">

&nbsp;

### Direct Communication

Company A announces its intention to raise 2 Billion USD by way of rights issue. Because this is a voluntary action, Company A needs to ensure that complete details of the offer are sent to shareholders in real time, regardless of how many intermediaries are involved in the process (such as receiving/paying agents, CSD, ICSD, local/global custodian banks, asset management firms, etc). Once a shareholder has made a decision, that decision will also be processed and settled (including the new issuance of shares) in real time. If a shareholder sold its rights to a third party, the securities depository must be able to record the new shares under the name of their new rightful owner.

<img src="../images/Canonical-Use-Cases_Direct-Communication.png" width="900" height="416">

&nbsp;

### Separation of Asset Ownership and Custodian’s Duties

Assets should always be owned by their actual owners, and asset owners must be able to allow third-party professionals to manage their assets without having to pass legal ownership of assets to third parties (such as nominee or street name entities). If issuers need to send messages or payments to asset owners (for example, listed share holders), issuers send them directly to asset owners. Third-party asset managers and/or custodians can always buy, sell, and lend assets on behalf of their owners. Under this arrangement, asset custodians can focus on providing value-add services to shareowners, without worrying about asset ownership duties such as managing and redirecting payments from issuers to shareowners.

<img src="../images/Canonical-Use-Cases_Separation-of-Asset-Ownership-and-Custodians-Duties.png" width="900" height="628">

&nbsp;

### Interoperability of Assets

If an organization requires 20,000 units of asset B, but instead owns 10,000 units of asset A, it needs a way to exchange asset A for asset B. Though the current market might not offer enough liquidity to fulfill this trade quickly, there might be plenty of liquidity available between asset A and asset C, and also between asset C and asset B. Instead of settling for market limits on direct trading (A for B) in this case, a chain network connects buyers with "buried" sellers, finds the best match (which could be buried under several layers of assets), and executes the transaction.

<img src="../images/Canonical-Use-Cases_Interoperability-of-Assets.png" width="900" height="480">
