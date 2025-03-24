> 前端日常使用的npm与pnpm有哪些不同，pnpm解决了哪些问题。他们又是怎么做依赖包多版本的问题的。以及node_modules过于庞大的时候，在CI/CD时会比较耗时，这又该如何解决

### npm 与 pnpm 的不同

1. **依赖管理**：
   
   - **npm**：每个项目都有独立的 `node_modules` 目录，依赖包会重复安装，导致磁盘空间浪费。
   
   - **pnpm**：使用全局存储（`~/.pnpm-store`），所有项目共享相同的依赖包，通过硬链接和符号链接引用，节省磁盘空间。

2. **安装速度**：
   
   - **npm**：安装依赖时，每个项目都需要重新下载和安装，速度较慢。
   
   - **pnpm**：依赖包只需下载一次，后续通过链接引用，安装速度更快。

3. **磁盘空间**：
   
   - **npm**：每个项目都有完整的依赖副本，占用大量磁盘空间。
   
   - **pnpm**：依赖包全局存储，项目通过链接引用，显著减少磁盘占用。

4. **依赖隔离**：
   
   - **npm**：依赖包可能因版本冲突导致问题。
   
   - **pnpm**：通过符号链接实现依赖隔离，避免冲突。

### pnpm 解决的问题

1. **磁盘空间浪费**：通过全局存储和链接机制，减少重复安装。

2. **安装速度慢**：依赖包只需下载一次，后续安装更快。

3. **依赖冲突**：通过符号链接隔离依赖，避免冲突。

### 依赖包多版本管理

1. **npm**：在 `node_modules` 中为每个依赖包创建独立目录，不同版本可以共存，但可能导致重复安装。

2. **pnpm**：依赖包存储在全局存储中，项目通过符号链接引用，不同版本的依赖包可以共存且不重复占用空间。

### 解决 CI/CD 中 `node_modules` 庞大导致的耗时问题

1. **缓存机制**：
   
   - **npm**：使用 `npm ci` 和 `npm cache` 减少安装时间。
   
   - **pnpm**：利用全局存储和缓存，依赖包只需下载一次，后续构建更快。

2. **依赖预安装**：
   
   - 在 CI/CD 前预安装依赖，构建时直接使用缓存，减少安装时间。

3. **使用更快的工具**：
   
   - 使用 `pnpm` 替代 `npm`，利用其全局存储和链接机制提升安装速度。

4. **减少依赖**：
   
   - 定期清理无用依赖，减少 `node_modules` 大小，提升构建速度。

---

> 1. 依赖包可能因版本冲突导致问题，请具体举例分析；
> 
> 2. pnpm针对不同版本的依赖包具体是怎么做的，多个版本都下载并储存在~/.pnpm-store中吗；
> 
> 3. pnpm存在哪些劣势；
> 
> 4. npm ci 的使用缓存和pnpm使用缓存区别在哪；

### 1. 依赖包可能因版本冲突导致问题，请具体举例分析

**依赖冲突**通常发生在项目依赖的多个包需要同一个依赖包的不同版本时。例如：

- **项目依赖**：
  
  - `packageA` 依赖 `lodash@4.17.20`
  
  - `packageB` 依赖 `lodash@3.10.1`

- **npm 的处理方式**：
  
  - npm 会将 `lodash` 安装在 `node_modules` 的根目录下，版本由依赖树决定。如果 `packageA` 和 `packageB` 对 `lodash` 的版本要求不兼容，可能会导致运行时错误。
  
  - 例如，`packageB` 可能使用了 `lodash@3.10.1` 的某个 API，但实际安装的是 `lodash@4.17.20`，导致运行时崩溃。

- **pnpm 的处理方式**：
  
  - pnpm 会将不同版本的 `lodash` 分别安装在 `~/.pnpm-store` 中，并通过符号链接在项目的 `node_modules` 中引用正确的版本。
  
  - 例如：
    
    - `node_modules/packageA/.pnpm/lodash@4.17.20` 链接到 `~/.pnpm-store/lodash@4.17.20`
    
    - `node_modules/packageB/.pnpm/lodash@3.10.1` 链接到 `~/.pnpm-store/lodash@3.10.1`
  
  - 这样，`packageA` 和 `packageB` 各自使用正确的 `lodash` 版本，避免了冲突。

### 2. pnpm 针对不同版本的依赖包具体是怎么做的，多个版本都下载并储存在 `~/.pnpm-store` 中吗？

是的，pnpm 会将不同版本的依赖包都下载并存储在全局存储目录 `~/.pnpm-store` 中。具体机制如下：

- **全局存储**：
  
  - 所有依赖包都会被下载到 `~/.pnpm-store` 中，每个版本的依赖包都有一个独立的目录。
  
  - 例如：
    
    - `~/.pnpm-store/lodash@4.17.20`
    
    - `~/.pnpm-store/lodash@3.10.1`

- **符号链接**：
  
  - 在项目的 `node_modules` 中，pnpm 会为每个依赖包创建符号链接，指向全局存储中的对应版本。
  
  - 例如：
    
    - `node_modules/packageA/.pnpm/lodash@4.17.20` 是一个符号链接，指向 `~/.pnpm-store/lodash@4.17.20`
    
    - `node_modules/packageB/.pnpm/lodash@3.10.1` 是一个符号链接，指向 `~/.pnpm-store/lodash@3.10.1`

- **依赖隔离**：
  
  - 每个包只能访问自己依赖的版本，避免了版本冲突。

### 3. pnpm 存在哪些劣势？

尽管 pnpm 有很多优点，但它也有一些劣势：

1. **兼容性问题**：
   
   - 某些工具或库可能假设 `node_modules` 的结构是 npm 或 Yarn 的标准结构，导致与 pnpm 不兼容。
   
   - 例如，某些工具可能直接遍历 `node_modules` 目录，而 pnpm 的符号链接结构可能导致它们无法正确找到依赖。

2. **学习成本**：
   
   - pnpm 的依赖管理机制与 npm 和 Yarn 不同，开发者需要适应新的工作流程。

3. **生态支持**：
   
   - 虽然 pnpm 的生态在逐渐完善，但某些工具或插件可能对 pnpm 的支持不如 npm 或 Yarn 成熟。

4. **符号链接的潜在问题**：
   
   - 在某些操作系统或文件系统上，符号链接可能会导致性能问题或兼容性问题。

### 4. `npm ci` 的使用缓存和 pnpm 使用缓存区别在哪？

- **`npm ci` 的缓存机制**：
  
  - `npm ci` 会使用 npm 的本地缓存（通常位于 `~/.npm` 目录）来加速安装。
  
  - 它会删除现有的 `node_modules`，然后根据 `package-lock.json` 重新安装依赖。
  
  - 如果依赖包已经在缓存中，`npm ci` 会直接从缓存中复制，而不是重新下载。
  
  - 但 `npm ci` 仍然会在每个项目的 `node_modules` 中创建完整的依赖副本，即使多个项目使用相同的依赖包。

- **pnpm 的缓存机制**：
  
  - pnpm 使用全局存储（`~/.pnpm-store`）来缓存所有依赖包。
  
  - 安装时，pnpm 会检查全局存储中是否已经存在所需的依赖包。如果存在，则通过硬链接或符号链接引用，而不是重新下载或复制。
  
  - 多个项目可以共享同一个全局存储，避免了重复下载和存储。

**主要区别**：

- `npm ci` 的缓存是项目级别的，每个项目仍然需要独立的 `node_modules`。

- pnpm 的缓存是全局的，多个项目可以共享同一个缓存，节省磁盘空间和安装时间。

----

### `package-lock.json` 的作用

1. **锁定依赖版本**：
   
   - `package-lock.json` 记录了项目中所有依赖包的确切版本号，包括直接依赖和间接依赖（依赖的依赖）。
   
   - 这样可以确保在不同环境（开发、测试、生产）中安装的依赖版本完全一致，避免因版本不一致导致的潜在问题。

2. **描述依赖树结构**：
   
   - `package-lock.json` 不仅记录了依赖包的版本，还描述了依赖树的结构，包括每个依赖包的依赖关系。
   
   - 这样可以确保依赖树的拓扑结构一致，避免因依赖解析顺序不同导致的安装差异。

3. **提升安装速度**：
   
   - 由于 `package-lock.json` 已经记录了依赖树的结构，npm 在安装时可以直接根据该文件解析依赖，而不需要重新计算依赖树，从而提升安装速度。

4. **支持确定性构建**：
   
   - 通过锁定依赖版本和结构，`package-lock.json` 确保了构建的确定性，即在不同时间、不同环境中运行 `npm install` 都会得到相同的结果。

### `package-lock.json` 的工作原理

#### 1. **生成 `package-lock.json`**

- 当运行 `npm install` 时，npm 会根据 `package.json` 中的依赖声明解析依赖树，并生成或更新 `package-lock.json`。

- 如果 `package-lock.json` 已经存在，npm 会优先根据它来安装依赖。

#### 2. **文件结构**

- `package-lock.json` 是一个 JSON 文件，其结构如下：
  
  ```json
  {
    "name": "example-project",
    "version": "1.0.0",
    "lockfileVersion": 3,
    "requires": true,
    "dependencies": {
      "lodash": {
        "version": "4.17.21",
        "resolved": "https://registry.npmjs.org/lodash/-/lodash-4.17.21.tgz",
        "integrity": "sha512-1234567890abcdef",
        "requires": {
          "some-package": "^1.0.0"
        }
      },
      "some-package": {
        "version": "1.0.1",
        "resolved": "https://registry.npmjs.org/some-package/-/some-package-1.0.1.tgz",
        "integrity": "sha512-0987654321fedcba"
      }
    }
  }
  ```

- 关键字段：
  
  - `version`：依赖包的确切版本。
  
  - `resolved`：依赖包的下载地址。
  
  - `integrity`：依赖包的完整性校验值（基于内容的哈希值），用于验证下载的包是否被篡改。
  
  - `requires`：该依赖包自身的依赖。

#### 3. **依赖解析**

- 当运行 `npm install` 时，npm 会按照以下步骤解析依赖：
  
  1. 检查 `package-lock.json` 是否存在。
  
  2. 如果存在，则根据 `package-lock.json` 中的记录安装依赖。
  
  3. 如果不存在，则根据 `package.json` 解析依赖树，生成 `package-lock.json` 并安装依赖。

#### 4. **更新依赖**

- 当手动修改 `package.json` 或运行 `npm install <package>` 时，npm 会更新 `package-lock.json` 以反映最新的依赖树。

- 例如：
  
  - 运行 `npm install lodash@latest` 会更新 `lodash` 的版本，并更新 `package-lock.json`。
  
  - 运行 `npm update` 会更新所有依赖包到符合 `package.json` 中版本范围的最新版本，并更新 `package-lock.json`。

#### 5. **锁定依赖树**

- `package-lock.json` 确保了依赖树的锁定，即使间接依赖的版本发生变化，也不会影响项目的依赖结构。

- 例如：
  
  - 如果 `packageA` 依赖 `lodash@^4.17.0`，而 `lodash` 发布了新版本 `4.18.0`，`package-lock.json` 仍然会锁定 `lodash` 的版本为 `4.17.21`，除非手动更新。

### `package-lock.json` 的使用场景

1. **团队协作**：
   
   - 在团队开发中，`package-lock.json` 可以确保所有开发者使用相同的依赖版本，避免因版本不一致导致的问题。

2. **CI/CD**：
   
   - 在 CI/CD 中，`package-lock.json` 可以确保每次构建时安装的依赖版本一致，避免因依赖版本变化导致的构建失败。

3. **生产环境**：
   
   - 在生产环境中，`package-lock.json` 可以确保部署的依赖版本与开发和测试环境一致，避免因依赖版本差异导致的运行时错误。

---

### `package-lock.json` 与 `npm-shrinkwrap.json` 的区别

- **`package-lock.json`**：
  
  - 由 npm 自动生成和管理。
  
  - 主要用于锁定开发环境的依赖。
  
  - 不会被发布到 npm 仓库。

- **`npm-shrinkwrap.json`**：
  
  - 功能与 `package-lock.json` 相同，但可以被发布到 npm 仓库。
  
  - 通常用于锁定生产环境的依赖。

---

### 总结

- **`package-lock.json` 的作用**：
  
  - 锁定依赖版本和依赖树结构，确保安装结果的一致性。
  
  - 提升安装速度，支持确定性构建。

- **工作原理**：
  
  - 记录依赖包的确切版本、下载地址和依赖关系。
  
  - 在 `npm install` 时优先根据 `package-lock.json` 安装依赖。

- **使用场景**：
  
  - 团队协作、CI/CD、生产环境部署。

通过正确使用 `package-lock.json`，可以有效避免依赖版本不一致导致的问题，提升项目的稳定性和可维护性。

---

### npm  ci

> npm clean-install

#### npm ci 安装确切的软件包版本

与 `npm install` 不同，`npm ci` 将软件包完全按照 **package-lock.json** 文件中指定的版本安装。这样可以确保项目中所有开发人员使用的依赖库版本相同，便于版本管理。

#### npm ci 不使用缓存

`npm install` 使用本地缓存来加快安装速度，但 `npm ci` 不使用该缓存。它直接从注册表下载软件包，确保安装的软件包与锁定文件中指定的软件包完全相同，避免了版本不匹配或冲突的情况。

#### npm ci 先删除 node_modules 目录

在安装软件包之前，`npm ci` 会删除 `node_modules` 目录，并从头开始安装依赖项。这有助于确保没有来自先前安装的过时依赖项。

#### npm ci 比 npm install 更快

由于它不使用缓存并安装确切的软件包版本，`npm ci` 可以比 `npm install` 更快。这在 CI/CD（持续集成/持续部署）流水线中尤其重要，其中速度和一致性非常关键。在某些情况下，`npm ci` 比 `npm install` 快两倍。

---

> npm ci 为什么比 npm i 快

`npm ci` 比 `npm install`（或 `npm i`）快的主要原因在于它的设计目标和执行逻辑不同。`npm ci` 是专门为 **持续集成/持续部署（CI/CD）** 环境优化的命令，它的执行方式更加严格和高效。

### 1. **删除现有 `node_modules`**

- **`npm ci`**：
  
  - 在安装依赖之前，`npm ci` 会先删除整个 `node_modules` 目录。
  
  - 这样可以确保安装环境是干净的，避免残留文件或旧版本依赖导致的问题。

- **`npm install`**：
  
  - `npm install` 会尝试复用现有的 `node_modules` 目录，可能会保留一些旧文件或未使用的依赖。
  
  - 这种复用机制虽然节省了部分安装时间，但也可能导致依赖树不一致或安装结果不可预测。

**为什么更快**：

- 删除 `node_modules` 后，`npm ci` 可以避免复杂的依赖树检查和冲突解决，直接从 `package-lock.json` 安装依赖，减少了计算和文件操作的开销。

### 2. **严格依赖 `package-lock.json`**

- **`npm ci`**：
  
  - `npm ci` 完全依赖 `package-lock.json` 来安装依赖，不会修改 `package-lock.json` 或 `package.json`。
  
  - 它要求 `package-lock.json` 必须存在，并且与 `package.json` 完全一致。

- **`npm install`**：
  
  - `npm install` 会根据 `package.json` 重新计算依赖树，并可能更新 `package-lock.json`。
  
  - 如果 `package-lock.json` 不存在，`npm install` 会生成一个新的。

**为什么更快**：

- `npm ci` 直接根据 `package-lock.json` 安装依赖，跳过了依赖树解析和版本计算的过程，节省了大量时间。

### 3. **跳过依赖版本兼容性检查**

- **`npm ci`**：
  
  - `npm ci` 不会检查依赖包的版本兼容性，而是严格按照 `package-lock.json` 中的版本安装。
  
  - 这样可以避免复杂的版本解析和冲突解决。

- **`npm install`**：
  
  - `npm install` 会检查 `package.json` 中的版本范围，并尝试安装符合条件的最新版本。
  
  - 这个过程可能涉及复杂的依赖树解析和冲突解决。

**为什么更快**：

- `npm ci` 跳过了版本兼容性检查和冲突解决，直接安装锁定版本，减少了计算和网络请求的开销。

### 4. **并行安装依赖**

- **`npm ci`**：
  
  - `npm ci` 在安装依赖时采用了并行安装的方式，可以同时下载和安装多个依赖包。

- **`npm install`**：
  
  - `npm install` 的安装过程是串行的，逐个下载和安装依赖包。

**为什么更快**：

- 并行安装充分利用了网络带宽和 CPU 资源，显著提升了安装速度。

### 5. **缓存机制**

- **`npm ci`**：
  
  - `npm ci` 会优先使用本地缓存（`~/.npm`）中的依赖包，避免重复下载。
  
  - 如果缓存中存在所需的依赖包，`npm ci` 会直接从缓存中复制，而不是重新下载。

- **`npm install`**：
  
  - `npm install` 也会使用缓存，但在某些情况下（如版本范围更新）可能会重新下载依赖包。

**为什么更快**：

- `npm ci` 的缓存机制更加高效，避免了不必要的网络请求。

### 6. **适合 CI/CD 环境**

- **`npm ci`**：
  
  - `npm ci` 是为 CI/CD 环境设计的，它的执行逻辑更加严格和高效。
  
  - 它确保了安装结果的确定性和一致性，适合自动化构建和部署。

- **`npm install`**：
  
  - `npm install` 更适合开发环境，允许灵活的依赖更新和版本解析。

**为什么更快**：

- `npm ci` 的设计目标决定了它的执行逻辑更加简洁和高效，避免了不必要的操作。

`npm ci` 比 `npm install` 快的原因可以归纳为以下几点：

1. **删除现有 `node_modules`**：确保干净的安装环境，避免冲突。

2. **严格依赖 `package-lock.json`**：跳过依赖树解析和版本计算。

3. **跳过版本兼容性检查**：避免复杂的版本解析和冲突解决。

4. **并行安装依赖**：充分利用网络带宽和 CPU 资源。

5. **高效的缓存机制**：优先使用本地缓存，避免重复下载。

6. **为 CI/CD 优化**：执行逻辑更加严格和高效。

因此，在 CI/CD 环境中，推荐使用 `npm ci` 而不是 `npm install`，以确保安装过程的快速、稳定和一致。
