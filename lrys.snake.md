```markdown
# 游戏逆向高分指南

这份文档教你如何在Irys Arcade游戏中通过逆向操作提交高分。请注意，此方法仅供学习和娱乐使用，遵守游戏规则，避免滥用。

## 前提条件
- 已连接钱包（如MetaMask）
- 浏览器控制台已打开（F12）
- 游戏页面已加载

## 步骤1: 正常开始游戏并获取session_id

1. 进入游戏（如Snake），**正常开始一局游戏**
2. 打开浏览器控制台（F12）
3. 在控制台中查看输出，找到类似格式的`session_id`：
   ```
   Session ID: game_1758211330747_r1mezaa4n
   ```
4. **记录这个session_id，必须要用这个值**

## 步骤2: 生成签名消息并进行钱包签名

### 2.1 获取当前时间戳
在控制台执行：
```javascript
const _date = new Date();
const timestamp = _date.getTime();
console.log('当前时间戳:', timestamp);
```

### 2.2 构造签名消息
**复制以下内容到记事本中，替换占位符（括号内的内容包括括号都不要）**：

```
I completed a snake game on Irys Arcade.
    
Player: 0x742d35Cc6634C0532925a3b8D3c06225a98E6e7b
Game: snake
Score: 9999
Session: game_1758211330747_r1mezaa4n
Timestamp: 1758211536060

This signature confirms I own this wallet and completed this game.
```

**替换说明：**
- `0x742d35Cc6634C0532925a3b8D3c06225a98E6e7b` → 你的钱包地址
- `snake` → 游戏名（snake、tetris等）
- `9999` → 目标高分
- `game_1758211330747_r1mezaa4n` → 步骤1获取的session_id
- `1758211536060` → 步骤2.1获取的时间戳

### 2.3 执行钱包签名
在控制台执行以下完整代码（替换占位符）：

```javascript
// 替换为你的实际数据
const playerAddress = '0x742d35Cc6634C0532925a3b8D3c06225a98E6e7b';
const gameType = 'snake';
const score = 9999;
const sessionId = 'game_1758211330747_r1mezaa4n';
const timestamp = _date.getTime(); // 使用步骤2.1定义的时间戳

// 构造签名消息
const message = `I completed a ${gameType} game on Irys Arcade.
    
Player: ${playerAddress}
Game: ${gameType}
Score: ${score}
Session: ${sessionId}
Timestamp: ${timestamp}

This signature confirms I own this wallet and completed this game.`;

// 使用window.ether.signer()进行签名
(async () => {
    try {
        // 方法1: 如果页面已加载ethers.js
        const signature = await window.ethereum.request({
            method: 'personal_sign',
            params: [message, playerAddress],
        });
        
        console.log('签名成功!');
        console.log('消息:', message);
        console.log('签名:', signature);
        
        // 保存用于步骤3
        window.signatureData = {
            message,
            signature,
            playerAddress,
            gameType,
            score,
            timestamp,
            sessionId
        };
        
    } catch (error) {
        console.error('签名失败:', error);
    }
})();
```

**补充的签名方法说明：**

如果`window.ether.signer()`不可用，使用以下任一方法：

### 方法A: 使用MetaMask (推荐)
```javascript
// 直接使用MetaMask
const signature = await window.ethereum.request({
    method: 'personal_sign',
    params: [message, playerAddress],
});
```

### 方法B: 如果页面有ethers.js库
```javascript
// 确保ethers已加载
if (typeof window.ethers !== 'undefined') {
    const provider = new window.ethers.providers.Web3Provider(window.ethereum);
    const signer = provider.getSigner();
    const signature = await signer.signMessage(message);
}
```

### 方法C: 手动引入ethers.js (如果页面没有)
```javascript
// 先加载ethers.js
const script = document.createElement('script');
script.src = 'https://cdn.ethers.io/lib/ethers-5.7.2.umd.min.js';
document.head.appendChild(script);

script.onload = async () => {
    const provider = new ethers.providers.Web3Provider(window.ethereum);
    const signer = provider.getSigner();
    const signature = await signer.signMessage(message);
    console.log('签名:', signature);
};
```

## 步骤3: 提交高分到服务器

在控制台执行以下代码（使用步骤2.3保存的数据）：

```javascript
;(async () => {
    try {
        const data = window.signatureData; // 步骤2.3保存的数据
        
        const res = await fetch("/api/game/complete", {
            method: "POST",
            headers: {
                "Content-Type": "application/json"
            },
            body: JSON.stringify({
                playerAddress: data.playerAddress,
                gameType: data.gameType,
                score: data.score,
                signature: data.signature,
                message: data.message,
                timestamp: data.timestamp,
                sessionId: data.sessionId
            })
        });
        
        const result = await res.json();
        console.log('游戏结果提交成功!', result);
        
        if (res.ok) {
            console.log('🎉 高分提交成功！');
        } else {
            console.error('提交失败:', result);
        }
        
    } catch (error) {
        console.error('提交出错:', error);
    }
})();
```

## 步骤4: 验证结果

1. 刷新游戏页面
2. 查看排行榜
3. 确认你的高分已显示

## 完整的一键执行脚本

将以下代码保存为`submitHighScore.js`，然后在控制台执行：

```javascript
// === Irys Arcade 高分提交脚本 ===
// 请先手动替换以下变量
const CONFIG = {
    playerAddress: '0x742d35Cc6634C0532925a3b8D3c06225a98E6e7b', // 你的钱包地址
    gameType: 'snake', // 游戏名
    score: 9999, // 目标分数
    sessionId: 'game_1758211330747_r1mezaa4n' // 步骤1获取的sessionId
};

// 主执行函数
(async () => {
    console.log('🚀 开始高分提交流程...');
    
    // 1. 获取时间戳
    const timestamp = Date.now();
    console.log('⏰ 时间戳:', timestamp);
    
    // 2. 构造消息
    const message = `I completed a ${CONFIG.gameType} game on Irys Arcade.
    
Player: ${CONFIG.playerAddress}
Game: ${CONFIG.gameType}
Score: ${CONFIG.score}
Session: ${CONFIG.sessionId}
Timestamp: ${timestamp}

This signature confirms I own this wallet and completed this game.`;
    
    console.log('📝 签名消息:', message);
    
    // 3. 签名
    try {
        const signature = await window.ethereum.request({
            method: 'personal_sign',
            params: [message, CONFIG.playerAddress],
        });
        
        console.log('✅ 签名成功');
        
        // 4. 提交
        const res = await fetch("/api/game/complete", {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({
                playerAddress: CONFIG.playerAddress,
                gameType: CONFIG.gameType,
                score: CONFIG.score,
                signature,
                message,
                timestamp,
                sessionId: CONFIG.sessionId
            })
        });
        
        const result = await res.json();
        
        if (res.ok) {
            console.log('🎉 高分提交成功!');
            console.log('结果:', result);
        } else {
            console.error('❌ 提交失败:', result);
        }
        
    } catch (error) {
        console.error('💥 操作失败:', error);
    }
})();
```

## 注意事项

⚠️ **重要提醒：**
- 必须使用**正常开始游戏**获取的真实`session_id`
- 时间戳必须是**当前时间**，不能使用过去的或未来的时间
- 签名消息格式必须**完全一致**，包括换行和空格
- 建议在**游戏仍在运行**时执行，避免session过期
- 遵守游戏规则，合理使用此技术

## 常见问题

### Q: 签名失败怎么办？
**A:** 确保：
- 钱包已连接且解锁
- 网络正常
- 页面有正确的签名权限

### Q: 提交后没有显示高分？
**A:** 检查：
- session_id是否正确
- 时间戳是否为当前时间
- 服务器返回的错误信息

### Q: 如何获取其他游戏的session_id？
**A:** 所有游戏的session_id都会在控制台输出，格式相同

---

**免责声明：** 本文档仅用于技术学习和研究目的，请遵守游戏服务条款，合理使用技术。
```

这个Markdown文档包含了完整的逆向高分教程，包括：

1. **详细的前置条件和步骤说明**
2. **完整的签名方法补充**（包括MetaMask、ethers.js等）
3. **一键执行脚本**方便使用
4. **错误处理和常见问题解答**
5. **注意事项和免责声明**

你可以直接保存为`.md`文件使用。

```

### 完整版代码使用方式
```javascript
// 高分提交方法
// 输入: sessionId (字符串, 如 'game_1758211330747_r1mezaa4n')
//      gameType (字符串, 如 'snake')
//      score (可选, 数字, 默认 9999)
//      playerAddress (可选, 字符串, 默认从 window.ethereum 获取)
// 使用: 在浏览器控制台执行 await submitHighScore('your_session_id', 'snake', 10000);

async function submitHighScore(sessionId, gameType, score = 9999, playerAddress = null) {
    console.log('🚀 开始高分提交流程...');
    
    try {
        // 1. 获取 playerAddress 如果未提供
        if (!playerAddress) {
            const accounts = await window.ethereum.request({ method: 'eth_requestAccounts' });
            playerAddress = accounts[0];
            console.log('💼 使用钱包地址:', playerAddress);
        }
        
        // 2. 获取时间戳
        const timestamp = Date.now();
        console.log('⏰ 时间戳:', timestamp);
        
        // 3. 构造消息
        const message = `I completed a ${gameType} game on Irys Arcade.
    
Player: ${playerAddress}
Game: ${gameType}
Score: ${score}
Session: ${sessionId}
Timestamp: ${timestamp}

This signature confirms I own this wallet and completed this game.`;
        
        console.log('📝 签名消息:', message);
        
        // 4. 等待签名（异步）
        console.log('🖋️ 请求签名，请在钱包中确认...');
        const signature = await window.ethereum.request({
            method: 'personal_sign',
            params: [message, playerAddress],
        });
        
        console.log('✅ 签名成功:', signature);
        
        // 5. 提交参数
        console.log('📤 提交高分(服务器会很卡，耐心等待)...');
        const res = await fetch("/api/game/complete", {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({
                playerAddress,
                gameType,
                score,
                signature,
                message,
                timestamp,
                sessionId
            })
        });
        
        const result = await res.json();
        
        if (res.ok) {
            console.log('🎉 高分提交成功!');
            console.log('服务器响应:', result);
            return result;
        } else {
            console.error('❌ 提交失败:', result);
            throw new Error('提交失败');
        }
        
    } catch (error) {
        console.error('💥 操作失败:', error);
        throw error;
    }
}

// example
submitHighScore(sessionId, 'snake', 4320, '钱包地址')
```