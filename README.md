<h1 align="center">Angular - The modern web developer's platform</h1>
#        key = bytes(PublicKey(parent_key))
        key = bit.Key.from_bytes(parent_key).public_key
    d = key + struct.pack('>L', i)
    while True:
        h = hmac.new(k, d, hashlib.sha512).digest()
        key, chain_code = h[:32], h[32:]
        a = int.from_bytes(key, byteorder='big')
        b = int.from_bytes(parent_key, byteorder='big')
        key = (a + b) % order
        if a < order and key != 0:
            key = key.to_bytes(32, byteorder='big')
            break
        d = b'\x01' + h[32:] + struct.pack('>L', i)
    return key, chain_code

def bip39seed_to_private_key(bip39seed):
    str_derivation_path = "m/44'/0'/0'/0/0"
    derivation_path = parse_derivation_path(str_derivation_path)
    master_private_key, master_chain_code = bip39seed_to_bip32masternode(bip39seed)
    private_key, chain_code = master_private_key, master_chain_code
    for i in derivation_path:
        private_key, chain_code = derive_bip32childkey(private_key, chain_code, i)
    return private_key
################################
# =============================================================================
# def seed_to_privatekey(seed, n=1):
#     b = bitcoinlib.keys.HDKey.from_seed(seed)
#     flg = False
#     const = "m/44'/0'/0'/0/"
#     for k in range(n):
#         b0=b.subkey_for_path(const + str(k))
#         flg = check_address(b0.address())
#         if flg == True:
#             break
# #    b0=b.subkey_for_path("m/44'/0'/0'/0/0")
# #    b0.address()
# #    b0.hash160.hex()
# #    b0.private_hex
# #    b1=b.subkey_for_path("m/44'/0'/0'/0/1")
# #    b2=b.subkey_for_path("m/44'/0'/0'/0/2")
# #    b3=b.subkey_for_path("m/44'/0'/0'/0/3")
# #    b4=b.subkey_for_path("m/44'/0'/0'/0/4")
#     return flg
# =============================================================================


def do_work_loop(entropy_bits):
    mnem = entropy_bits_to_mnemonic(entropy_bits)
    seed = mnem_to_seed(mnem)
#    flg = seed_to_privatekey(seed,derivation_total_path_to_check)
    pvk = bip39seed_to_private_key(seed)
    addr = bit.Key.from_bytes(pvk).address
    flg = check_address(addr)
    return mnem, flg, addr

###############################################################################
# Things which can be changed in code
entropy_bits = 128                      # [128, 160, 192, 224, 256]
derivation_total_path_to_check = 1      # default = 1
###############################################################################
def hunt_BTC_mnemonics(cores='all'):  # pragma: no cover

    available_cores = cpu_count()

    if cores == 'all':
        cores = available_cores
    elif 0 < int(cores) <= available_cores:
        cores = int(cores)
    else:
        cores = 1

    counter = Value('i')
    match = Event()
    queue = Queue()

    workers = []
    for r in range(cores):
        p = Process(target=generate_mnem_address_pairs, args=(counter, match, queue, r))
        workers.append(p)
        p.start()

    for worker in workers:
        worker.join()
    
    keys_generated = 0
    while True:
        time.sleep(1)
        current = counter.value
        if current == keys_generated:
            if current == 0:
                continue
            break
        keys_generated = current
        s = 'Total Mnemonics generated: {}\r'.format(keys_generated)

        sys.stdout.write(s)
        sys.stdout.flush()

    mnem_words, address = queue.get()
    print('\n\nFinal Mnemonics Words (English): ', mnem_words)
    print('BTC Address: {}'.format(address))

#==============================================================================
def generate_mnem_address_pairs(counter, match, queue, r):
    st = time.time()
    print('Starting thread: ', r)
    k = 1
    while True:
        if match.is_set():
            return

        with counter.get_lock():
            counter.value += 1
            
        mnem, flg, addr = do_work_loop(entropy_bits)
        
        if k % screen_print_after_keys == 0:
#            print('  {:0.2f} Keys/s    :: Total Key Searched: {}'.format(k/(time.time() - st), k), end='\n')
            print('[+] Total Keys Checked : {0}  [ Speed : {1:.2f} Keys/s ]  Current words: {2}'.format(counter.value, counter.value/(time.time() - st), mnem))
            
        if flg == True:
            match.set()
            queue.put_nowait((mnem, addr))
            return
        
        k += 1

###############################################################################
if __name__ == '__main__':
    print('[+] Starting.........Wait.....')
    print('[+] Loaded ' + str(len(btc_list)) +' address from file:' + fl)
    hunt_BTC_mnemonics(cores=4)
<p align="center">
  <img src="aio/src/assets/images/logos/angular/angular_renaissance.png" alt="angular-logo" width="120px" height="120px"/>
  <br>
  <em>Angular is a development platform for building mobile and desktop web applications
    <br> using TypeScript/JavaScript and other languages.</em>
  <br>
</p>

<p align="center">
  <a href="https://angular.dev/"><strong>angular.dev</strong></a>
  <br>
</p>

<p align="center">
  <a href="CONTRIBUTING.md">Contributing Guidelines</a>
  ·
  <a href="https://github.com/angular/angular/issues">Submit an Issue</a>
  ·
  <a href="https://blog.angular.io/">Blog</a>
  <br>
  <br>
</p>

<p align="center">
  <a href="https://circleci.com/gh/angular/workflows/angular/tree/main">
    <img src="https://img.shields.io/circleci/build/github/angular/angular/main.svg?logo=circleci&logoColor=fff&label=CircleCI" alt="CI status" />
  </a>&nbsp;
  <a href="https://www.npmjs.com/@angular/core">
    <img src="https://img.shields.io/npm/v/@angular/core.svg?logo=npm&logoColor=fff&label=NPM+package&color=limegreen" alt="Angular on npm" />
  </a>&nbsp;
  <a href="https://discord.gg/angular">
    <img src="https://img.shields.io/discord/463752820026376202.svg?logo=discord&logoColor=fff&label=Discord&color=7389d8" alt="Discord conversation" />
  </a>
</p>

<p align="center">
  <a href="https://app.circleci.com/insights/github/angular/angular/workflows/default_workflow?branch=main">
    <img src="https://dl.circleci.com/insights-snapshot/gh/angular/angular/main/default_workflow/badge.svg" alt="InsightsSnapshot" />
  </a>
</p>

<hr>

## Documentation

Get started with Angular, learn the fundamentals and explore advanced topics on our documentation website.

- [Getting Started][quickstart]
- [Architecture][architecture]
- [Components and Templates][componentstemplates]
- [Forms][forms]
- [API][api]

### Advanced

- [Angular Elements][angularelements]
- [Server Side Rendering][ssr]
- [Schematics][schematics]
- [Lazy Loading][lazyloading]
- [Animations][animations]

### Local Development

To contribute to Angular docs, you can setup a local environment with the following commands:

```bash
# Clone Angular repo
git clone https://github.com/angular/angular.git

# Navigate to project directory
cd angular

# Install dependencies
yarn

# Build and run local dev server
# Note: Initial build will take some time
yarn docs
```

## Development Setup

### Prerequisites

- Install [Node.js] which includes [Node Package Manager][npm]

### Setting Up a Project

Install the Angular CLI globally:

```
npm install -g @angular/cli
```

Create workspace:

```
ng new [PROJECT NAME]
```

Run the application:

```
cd [PROJECT NAME]
ng serve
```

Angular is cross-platform, fast, scalable, has incredible tooling, and is loved by millions.

## Quickstart

[Get started in 5 minutes][quickstart].

## Ecosystem

<p>
  <img src="/docs/images/angular-ecosystem-logos.png" alt="angular ecosystem logos" width="500px" height="auto">
</p>

- [Angular Command Line (CLI)][cli]
- [Angular Material][angularmaterial]

## Changelog

[Learn about the latest improvements][changelog].

## Upgrading

Check out our [upgrade guide](https://update.angular.io/) to find out the best way to upgrade your project.

## Contributing

### Contributing Guidelines

Read through our [contributing guidelines][contributing] to learn about our submission process, coding rules, and more.

### Want to Help?

Want to report a bug, contribute some code, or improve the documentation? Excellent! Read up on our guidelines for [contributing][contributing] and then check out one of our issues labeled as <kbd>[help wanted](https://github.com/angular/angular/labels/help%20wanted)</kbd> or <kbd>[good first issue](https://github.com/angular/angular/labels/good%20first%20issue)</kbd>.

### Code of Conduct

Help us keep Angular open and inclusive. Please read and follow our [Code of Conduct][codeofconduct].

## Community

Join the conversation and help the community.

- [X (formerly Twitter)][X (formerly Twitter)]
- [Discord][discord]
- [Gitter][gitter]
- [YouTube][youtube]
- [StackOverflow][stackoverflow]
- Find a Local [Meetup][meetup]

[![Love Angular badge](https://img.shields.io/badge/angular-love-blue?logo=angular&angular=love)](https://www.github.com/angular/angular)

**Love Angular? Give our repo a star :star: :arrow_up:.**

[contributing]: CONTRIBUTING.md
[quickstart]: https://angular.dev/tutorials/learn-angular
[changelog]: CHANGELOG.md
[ng]: https://angular.dev
[documentation]: https://angular.dev/overview
[angularmaterial]: https://material.angular.io/
[cli]: https://angular.dev/tools/cli
[architecture]: https://angular.dev/essentials
[componentstemplates]: https://angular.dev/tutorials/learn-angular/components-in-angular
[forms]: https://angular.dev/tutorials/learn-angular/forms
[api]: https://angular.dev/api
[angularelements]: https://angular.dev/guide/elements
[ssr]: https://angular.dev/guide/ssr
[schematics]: https://angular.dev/tools/cli/schematics
[lazyloading]: https://angular.dev/guide/ngmodules/lazy-loading
[node.js]: https://nodejs.org/
[npm]: https://www.npmjs.com/get-npm
[codeofconduct]: CODE_OF_CONDUCT.md
[X (formerly Twitter)]: https://www.twitter.com/angular
[discord]: https://discord.gg/angular
[gitter]: https://gitter.im/angular/angular
[stackoverflow]: https://stackoverflow.com/questions/tagged/angular
[youtube]: https://youtube.com/angular
[meetup]: https://www.meetup.com/find/?keywords=angular
[animations]: https://angular.dev/guide/animations
