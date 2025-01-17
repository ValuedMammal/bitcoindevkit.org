---
cases: true
sidebar: false
tagline: "Bitcoin applications building with BDK"
description: "A list of bitcoin applications and services building with BDK"
actionText: "Add your project"
actionLink: "https://github.com/orgs/bitcoindevkit/discussions/64"
editLink: false
lastUpdated: false
---

<h1 class="more-cases-heading">
   Meet the projects building with BDK
</h1>

<!-- <CodeSwitcher :languages="{all: 'All', mobile:'Mobile', web:'Web', desktop:'Desktop', custodial: 'Custodial', infra:'Infrastructure', misc:'Misc',}"> -->
  
<CodeSwitcher :languages="{ all: 'All', mobile: 'Mobile', desktop: 'Desktop', hardware: 'Hardware', web:'Web', custodial: 'Custodial', exchange: 'Exchange' }">

  <template v-slot:mobile>
    <div class="case-studies">
      <div class="case-study-item">
        <a href="https://bitkey.build/" target="_blank">
          <img src="/img/case-studies-logos/block-logo.gif" style="max-height: 130px;" />
        </a>
        <h3>
          <a href="https://bitkey.build/" target="_blank">Bitkey</a> 
        </h3>
        <p>Bitkey is the safe, easy way to own and manage bitcoin. It’s a mobile app, hardware device, and a set of recovery tools, for simple, secure self-custody.</p>
      </div>
      <div class="case-study-item">
        <a href="https://peachbitcoin.com/" target="_blank">
          <img src="/img/case-studies-logos/peach-130.png" />
        </a>
        <h3>
          <a href="https://peachbitcoin.com/" target="_blank">Peach Bitcoin</a>
        </h3>
        <p>Connecting Bitcoin buyers and sellers directly together. Buy or sell bitcoin peer-to-peer anywhere, at anytime.</p>
      </div>
      <div class="case-study-item">
        <a href="https://github.com/lightningdevkit/ldk-node" target="_blank">
          <img src="/img/case-studies-logos/ldk-node-130.png" />
        </a>
        <h3>
          <a href="https://github.com/lightningdevkit/ldk-node" target="_blank">LDK Node</a> 
        </h3>
        <p>A ready-to-go Lightning node library built using LDK and BDK.</p>
      </div>
      <div class="case-study-item">
        <a href="https://www.lava.xyz/" target="_blank">
          <img src="/img/case-studies-logos/lava-130.png" />
        </a>
        <h3>
          <a href="https://www.lava.xyz/" target="_blank">Lava</a>
        </h3>
        <p>The Future of Finance Available Today. Functional, safe and simple.</p>
      </div>
      <div class="case-study-item">
        <a href="https://play.google.com/store/apps/details?id=com.goldenraven.padawanwallet" target="_blank">
          <img src="/img/case-studies-logos/padawan-130.png" />
        </a>
        <h3>
          <a href="https://play.google.com/store/apps/details?id=com.goldenraven.padawanwallet" target="_blank">Padawan Wallet</a>
        </h3>
        <p>Padawan is a testnet-only bitcoin wallet packed with tutorials to learn how to use bitcoin on mobile.</p>
      </div>
      <div class="case-study-item">
        <a href="https://www.mutinywallet.com/" target="_blank">
          <img src="/img/case-studies-logos/mutiny-130.png" />
        </a>
        <h3>
          <a href="https://www.mutinywallet.com/" target="_blank">Mutiny Wallet</a>
        </h3>
        <p>Mutiny is a self-custodial lightning wallet that runs in the browser.</p>
      </div>
      <div class="case-study-item">
        <a href="https://foundationdevices.com/" target="_blank">
          <img src="/img/case-studies-logos/foundation-130.png" />
        </a>
        <h3>
          <a href="https://foundationdevices.com/" target="_blank">Envoy By Foundation</a> 
        </h3>
        <p>A Bitcoin wallet with powerful account management and privacy features. Use alongside your Passport hardware wallet to take true ownership of your Bitcoin.</p>
      </div>
      <div class="case-study-item">
        <a href="https://www.bullbitcoin.com/" target="_blank">
          <img src="/img/case-studies-logos/bull-bitcoin-130.png" />
        </a>
        <h3>
          <a href="https://www.bullbitcoin.com/" target="_blank">Bull Bitcoin</a>
        </h3>
        <p>A self-custodial Bitcoin Wallet and Exchange app that lets users buy, sell, spend and get paid with Bitcoin. Bitcoins are automatically sent from the exchange to the user's wallet.</p>
      </div>
    </div>
  </template>

  <template v-slot:exchange>
    <div class="case-studies">
      <div class="case-study-item">
        <a href="https://www.bullbitcoin.com/" target="_blank">
          <img src="/img/case-studies-logos/bull-bitcoin-130.png" />
        </a>
        <h3>
          <a href="https://www.bullbitcoin.com/" target="_blank">Bull Bitcoin</a>
        </h3>
        <p>A self-custodial Bitcoin Wallet and Exchange app that lets users buy, sell, spend and get paid with Bitcoin. Bitcoins are automatically sent from the exchange to the user's wallet.</p>
      </div>
    </div>
  </template>

  <template v-slot:desktop>
    <div class="case-studies">
      <div class="case-study-item">
        <a href="https://www.anchorwatch.com/" target="_blank">
          <img src="/img/case-studies-logos/anchorwatch-130.png" />
        </a>
        <h3>
          <a href="https://www.anchorwatch.com/" target="_blank">AnchorWatch</a>
        </h3>
        <p>Protect your bitcoin with regulated insurance and enterprise-grade multi-institutional custody.</p>
      </div>
    </div>
  </template>

  <template v-slot:hardware>
    <div class="case-studies">
      <div class="case-study-item">
        <a href="https://bitkey.build/" target="_blank">
          <img src="/img/case-studies-logos/block-logo.gif" style="max-height: 130px;" />
        </a>
        <h3>
          <a href="https://bitkey.build/" target="_blank">Bitkey</a> 
        </h3>
        <p>Bitkey is the safe, easy way to own and manage bitcoin. It’s a mobile app, hardware device, and a set of recovery tools, for simple, secure self-custody.</p>
      </div>
      <div class="case-study-item">
        <a href="https://foundationdevices.com/" target="_blank">
          <img src="/img/case-studies-logos/foundation-130.png" />
        </a>
        <h3>
          <a href="https://foundationdevices.com/" target="_blank">Envoy By Foundation</a> 
        </h3>
        <p>A Bitcoin wallet with powerful account management and privacy features. Use alongside your Passport hardware wallet to take true ownership of your Bitcoin.</p>
      </div>
    </div>
  </template>

  <template v-slot:custodial>
    <div class="case-studies">
      <div class="case-study-item">
        <a href="https://www.seba.swiss/" target="_blank">
          <img src="/img/case-studies-logos/seba-130.png" />
        </a>
        <h3>
          <a href="https://www.seba.swiss/" target="_blank">Seba Bank</a>
        </h3>
        <p>From everyday banking to crypto custody and trading, get the most out of your assets with a regulated global crypto bank.</p>
      </div>
    </div>
  </template>

  <!-- <template v-slot:infra>
    <div class="case-studies">
      <div class="case-study-item">
        <a href="https://bitcoindevkit.org" target="_blank">
          <img src="/img/bitcoindevkit.svg" />
        </a>
        <h3>
          <a href="https://bitcoindevkit.org" target="_blank">Example Infrastructure App</a>
        </h3>
        <p>A cool app built with BDK.</p>
      </div>
    </div>
  </template> -->

  <template v-slot:web>
    <div class="case-studies">
      <div class="case-study-item">
        <a href="https://bitmask.app/" target="_blank">
          <img src="/img/case-studies-logos/bitmask-130.png" />
        </a>
        <h3>
          <a href="https://bitmask.app/" target="_blank">BitMask Wallet</a>
        </h3>
        <p>Your Gateway to DeepWeb3 on Bitcoin. A browser extension for decentralized applications on Bitcoin.</p>
      </div>
    </div>
  </template>

  <template v-slot:all>
    <div class="case-studies">
      <div class="case-study-item">
        <a href="https://bitkey.build/" target="_blank">
          <img src="/img/case-studies-logos/block-logo.gif" style="max-height: 130px;" />
        </a>
        <h3>
          <a href="https://bitkey.build/" target="_blank">Bitkey</a> 
        </h3>
        <p>Bitkey is the safe, easy way to own and manage bitcoin. It’s a mobile app, hardware device, and a set of recovery tools, for simple, secure self-custody.</p>
      </div>
      <div class="case-study-item">
        <a href="" target="_blank">
          <img src="/img/case-studies-logos/peach-130.png" />
        </a>
        <h3>
          <a href="https://peachbitcoin.com/" target="_blank">Peach Bitcoin</a>
        </h3>
        <p>Connecting Bitcoin buyers and sellers directly together. Buy or sell bitcoin peer-to-peer anywhere, at anytime.</p>
      </div>
      <div class="case-study-item">
        <a href="https://www.anchorwatch.com/" target="_blank">
          <img src="/img/case-studies-logos/anchorwatch-130.png" />
        </a>
        <h3>
          <a href="https://www.anchorwatch.com/" target="_blank">AnchorWatch</a>
        </h3>
        <p>Protect your bitcoin with regulated insurance and enterprise-grade multi-institutional custody.</p>
      </div>
      <div class="case-study-item">
        <a href="https://www.mutinywallet.com/" target="_blank">
          <img src="/img/case-studies-logos/mutiny-130.png" />
        </a>
        <h3>
          <a href="https://www.mutinywallet.com/" target="_blank">Mutiny Wallet</a>
        </h3>
        <p>Mutiny is a self-custodial lightning wallet that runs in the browser.</p>
      </div>
      <div class="case-study-item">
        <a href="https://foundationdevices.com/" target="_blank">
          <img src="/img/case-studies-logos/foundation-130.png" />
        </a>
        <h3>
          <a href="https://foundationdevices.com/" target="_blank">Envoy By Foundation</a> 
        </h3>
        <p>A Bitcoin wallet with powerful account management and privacy features. Use alongside your Passport hardware wallet to take true ownership of your Bitcoin.</p>
      </div>
            <div class="case-study-item">
        <a href="https://www.bullbitcoin.com/" target="_blank">
          <img src="/img/case-studies-logos/bull-bitcoin-130.png" />
        </a>
        <h3>
          <a href="https://www.bullbitcoin.com/" target="_blank">Bull Bitcoin</a>
        </h3>
        <p>A self-custodial Bitcoin Wallet and Exchange app that lets users buy, sell, spend and get paid with Bitcoin. Bitcoins are automatically sent from the exchange to the user's wallet.</p>
      </div>
      <div class="case-study-item">
        <a href="https://www.lava.xyz/" target="_blank">
          <img src="/img/case-studies-logos/lava-130.png" />
        </a>
        <h3>
          <a href="https://www.lava.xyz/" target="_blank">Lava</a>
        </h3>
        <p>The Future of Finance Available Today. Functional, safe and simple.</p>
      </div>
      <div class="case-study-item">
        <a href="https://github.com/lightningdevkit/ldk-node" target="_blank">
          <img src="/img/case-studies-logos/ldk-node-130.png" />
        </a>
        <h3>
          <a href="https://github.com/lightningdevkit/ldk-node" target="_blank">LDK Node</a> 
        </h3>
        <p>A ready-to-go Lightning node library built using LDK and BDK.</p>
      </div>
      <div class="case-study-item">
        <a href="https://play.google.com/store/apps/details?id=com.goldenraven.padawanwallet" target="_blank">
          <img src="/img/case-studies-logos/padawan-130.png" />
        </a>
        <h3>
          <a href="https://play.google.com/store/apps/details?id=com.goldenraven.padawanwallet" target="_blank">Padawan Wallet</a>
        </h3>
        <p>Padawan is a testnet-only bitcoin wallet packed with tutorials to learn how to use bitcoin on mobile.</p>
      </div>
      <div class="case-study-item">
        <a href="https://www.seba.swiss/" target="_blank">
          <img src="/img/case-studies-logos/seba-130.png" />
        </a>
        <h3>
          <a href="https://www.seba.swiss/" target="_blank">Seba Bank</a>
        </h3>
        <p>From everyday banking to crypto custody and trading, get the most out of your assets with a regulated global crypto bank.</p>
      </div>
      <div class="case-study-item">
        <a href="https://bitmask.app/" target="_blank">
          <img src="/img/case-studies-logos/bitmask-130.png" />
        </a>
        <h3>
          <a href="https://bitmask.app/" target="_blank">BitMask Wallet</a>
        </h3>
        <p>Your Gateway to DeepWeb3 on Bitcoin. A browser extension for decentralized applications on Bitcoin.</p>
      </div>
    </div>
  </template>

</CodeSwitcher>
