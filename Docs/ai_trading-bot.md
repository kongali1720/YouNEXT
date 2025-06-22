// PHASE 2: AI Trading Bot with Machine Learning
// Advanced automated trading system for YouNEXT Platform

class AITradingBot {
    constructor(platform) {
        this.platform = platform;
        this.isActive = false;
        this.strategies = new Map();
        this.models = new Map();
        this.tradingHistory = [];
        this.riskManager = new RiskManager();
        this.portfolioManager = new PortfolioManager();
        this.signalGenerator = new SignalGenerator();
        this.executionEngine = new ExecutionEngine(platform);
        this.performanceTracker = new PerformanceTracker();
        this.mlPredictor = new MLPredictor();
        
        // Bot configuration
        this.config = {
            maxRiskPerTrade: 0.02, // 2% max risk per trade
            maxDailyLoss: 0.05,    // 5% max daily loss
            maxOpenPositions: 5,    // Max simultaneous positions
            minConfidenceLevel: 0.7, // Minimum signal confidence
            rebalanceInterval: 60000, // 1 minute
            stopLossPercent: 0.03,  // 3% stop loss
            takeProfitPercent: 0.06, // 6% take profit
            trailingStopPercent: 0.02, // 2% trailing stop
            enableMartingale: false,
            enableGridTrading: true,
            enableDCAStrategy: true,
            emergencyStopEnabled: true
        };
        
        // AI Models
        this.aiModels = {
            pricePredictor: new PricePredictionModel(),
            sentimentAnalyzer: new SentimentAnalysisModel(),
            riskAssessor: new RiskAssessmentModel(),
            patternRecognizer: new PatternRecognitionModel(),
            volatilityPredictor: new VolatilityPredictionModel()
        };
        
        // Active positions
        this.positions = new Map();
        this.orders = new Map();
        this.alerts = [];
        
        this.init();
    }
    
    init() {
        console.log('🤖 Initializing AI Trading Bot...');
        
        this.initializeStrategies();
        this.initializeModels();
        this.setupEventListeners();
        this.startMonitoring();
        
        console.log('✅ AI Trading Bot initialized successfully!');
    }
    
    // Initialize trading strategies
    initializeStrategies() {
        // Trend Following Strategy
        this.strategies.set('trendFollowing', {
            name: 'Trend Following',
            description: 'Follows market trends using moving averages and momentum',
            enabled: true,
            parameters: {
                fastMA: 20,
                slowMA: 50,
                rsi: { period: 14, overbought: 70, oversold: 30 },
                macd: { fast: 12, slow: 26, signal: 9 },
                volumeThreshold: 1.5,
                trendStrength: 0.6
            },
            performance: { winRate: 0, totalTrades: 0, profit: 0 }
        });
        
        // Mean Reversion Strategy
        this.strategies.set('meanReversion', {
            name: 'Mean Reversion',
            description: 'Exploits price reversions to statistical mean',
            enabled: true,
            parameters: {
                bollingerPeriod: 20,
                bollingerDeviation: 2,
                rsiPeriod: 14,
                rsiOversold: 30,
                rsioverbought: 70,
                zScoreThreshold: 2,
                reentryZScore: 1
            },
            performance: { winRate: 0, totalTrades: 0, profit: 0 }
        });
        
        // Breakout Strategy
        this.strategies.set('breakout', {
            name: 'Breakout Trading',
            description: 'Identifies and trades price breakouts',
            enabled: true,
            parameters: {
                lookbackPeriod: 20,
                volumeMultiplier: 2,
                breakoutThreshold: 0.02,
                falseBreakoutFilter: 0.5,
                consolidationPeriod: 10
            },
            performance: { winRate: 0, totalTrades: 0, profit: 0 }
        });
        
        // Arbitrage Strategy
        this.strategies.set('arbitrage', {
            name: 'Statistical Arbitrage',
            description: 'Exploits price differences between correlated assets',
            enabled: false, // Disabled by default (requires multiple exchanges)
            parameters: {
                correlationThreshold: 0.8,
                spreadThreshold: 0.01,
                halfLife: 24,
                lookbackWindow: 100
            },
            performance: { winRate: 0, totalTrades: 0, profit: 0 }
        });
        
        // Scalping Strategy
        this.strategies.set('scalping', {
            name: 'AI Scalping',
            description: 'High-frequency micro-profit trading',
            enabled: false, // High risk strategy
            parameters: {
                timeframe: '1m',
                targetProfit: 0.002, // 0.2%
                stopLoss: 0.001,     // 0.1%
                maxHoldTime: 300000, // 5 minutes
                volumeFilter: true
            },
            performance: { winRate: 0, totalTrades: 0, profit: 0 }
        });
        
        // DCA (Dollar Cost Averaging) Strategy
        this.strategies.set('dca', {
            name: 'DCA Strategy',
            description: 'Systematic dollar cost averaging with AI timing',
            enabled: true,
            parameters: {
                investmentAmount: 100,
                frequency: 'daily',
                priceDropThreshold: 0.05,
                volatilityAdjustment: true,
                sentimentFilter: true
            },
            performance: { winRate: 0, totalTrades: 0, profit: 0 }
        });
        
        console.log(`📊 Initialized ${this.strategies.size} trading strategies`);
    }
    
    // Initialize AI models
    initializeModels() {
        // Price prediction model
        this.aiModels.pricePredictor.initialize({
            inputFeatures: ['price', 'volume', 'rsi', 'macd', 'bb', 'sentiment'],
            outputHorizons: ['1h', '4h', '1d'],
            modelType: 'LSTM',
            layers: [64, 32, 16],
            activation: 'relu',
            epochs: 100,
            batchSize: 32
        });
        
        // Sentiment analysis model
        this.aiModels.sentimentAnalyzer.initialize({
            sources: ['twitter', 'reddit', 'news', 'telegram'],
            languages: ['en', 'id'],
            modelType: 'BERT',
            confidenceThreshold: 0.75
        });
        
        // Risk assessment model
        this.aiModels.riskAssessor.initialize({
            riskFactors: ['volatility', 'correlation', 'liquidity', 'market_cap'],
            timeHorizons: ['1h', '1d', '1w'],
            riskLevels: ['low', 'medium', 'high', 'extreme']
        });
        
        console.log('🧠 AI models initialized successfully');
    }
    
    // Start the trading bot
    async startBot() {
        if (this.isActive) {
            console.log('⚠️ Bot is already running');
            return;
        }
        
        try {
            console.log('🚀 Starting AI Trading Bot...');
            
            // Safety checks
            await this.performSafetyChecks();
            
            // Initialize models with current market data
            await this.trainModels();
            
            // Start trading loop
            this.isActive = true;
            this.startTradingLoop();
            
            // Start monitoring systems
            this.startRiskMonitoring();
            this.startPerformanceTracking();
            
            this.platform.showNotification('🤖 AI Trading Bot started successfully!', 'success');
            console.log('✅ AI Trading Bot is now active');
            
        } catch (error) {
            console.error('❌ Failed to start trading bot:', error);
            this.platform.showNotification('Failed to start trading bot: ' + error.message, 'error');
        }
    }
    
    // Stop the trading bot
    async stopBot() {
        if (!this.isActive) {
            console.log('⚠️ Bot is not running');
            return;
        }
        
        try {
            console.log('🛑 Stopping AI Trading Bot...');
            
            this.isActive = false;
            
            // Close all open positions (if configured)
            if (this.config.closePositionsOnStop) {
                await this.closeAllPositions();
            }
            
            // Cancel all pending orders
            await this.cancelAllOrders();
            
            // Save performance data
            this.savePerformanceData();
            
            this.platform.showNotification('🛑 AI Trading Bot stopped successfully!', 'info');
            console.log('✅ AI Trading Bot stopped');
            
        } catch (error) {
            console.error('❌ Error stopping trading bot:', error);
            this.platform.showNotification('Error stopping bot: ' + error.message, 'error');
        }
    }
    
    // Main trading loop
    async startTradingLoop() {
        console.log('🔄 Starting trading loop...');
        
        const tradingLoop = async () => {
            if (!this.isActive) return;
            
            try {
                // Update market data
                await this.updateMarketData();
                
                // Generate trading signals
                const signals = await this.generateTradingSignals();
                
                // Process signals and execute trades
                await this.processSignals(signals);
                
                // Update positions and orders
                await this.updatePositions();
                
                // Risk management checks
                await this.performRiskChecks();
                
                // Portfolio rebalancing
                await this.rebalancePortfolio();
                
            } catch (error) {
                console.error('❌ Error in trading loop:', error);
                this.handleTradingError(error);
            }
            
            // Schedule next iteration
            if (this.isActive) {
                setTimeout(tradingLoop, this.config.rebalanceInterval);
            }
        };
        
        // Start the loop
        tradingLoop();
    }
    
    // Generate trading signals using AI
    async generateTradingSignals() {
        const signals = [];
        const marketData = this.platform.marketData;
        
        for (const [symbol, market] of marketData) {
            try {
                // Skip if insufficient data
                if (!this.hasMinimumData(symbol)) continue;
                
                // Get technical indicators
                const indicators = await this.calculateIndicators(symbol, market);
                
                // Get sentiment analysis
                const sentiment = await this.analyzeSentiment(symbol);
                
                // Get price predictions
                const predictions = await this.predictPrice(symbol, indicators);
                
                // Generate signals for each strategy
                for (const [strategyName, strategy] of this.strategies) {
                    if (!strategy.enabled) continue;
                    
                    const signal = await this.generateStrategySignal(
                        strategyName, 
                        symbol, 
                        market, 
                        indicators, 
                        sentiment, 
                        predictions
                    );
                    
                    if (signal && signal.confidence >= this.config.minConfidenceLevel) {
                        signals.push(signal);
                    }
                }
                
            } catch (error) {
                console.error(`❌ Error generating signals for ${symbol}:`, error);
            }
        }
        
        // Sort signals by confidence and expected return
        signals.sort((a, b) => (b.confidence * b.expectedReturn) - (a.confidence * a.expectedReturn));
        
        console.log(`📊 Generated ${signals.length} trading signals`);
        return signals;
    }
    
    // Generate signal for specific strategy
    async generateStrategySignal(strategyName, symbol, market, indicators, sentiment, predictions) {
        const strategy = this.strategies.get(strategyName);
        if (!strategy) return null;
        
        let signal = null;
        
        switch (strategyName) {
            case 'trendFollowing':
                signal = this.generateTrendFollowingSignal(symbol, market, indicators, strategy.parameters);
                break;
                
            case 'meanReversion':
                signal = this.generateMeanReversionSignal(symbol, market, indicators, strategy.parameters);
                break;
                
            case 'breakout':
                signal = this.generateBreakoutSignal(symbol, market, indicators, strategy.parameters);
                break;
                
            case 'dca':
                signal = this.generateDCASignal(symbol, market, sentiment, strategy.parameters);
                break;
                
            case 'scalping':
                signal = this.generateScalpingSignal(symbol, market, indicators, strategy.parameters);
                break;
        }
        
        if (signal) {
            // Enhance signal with AI predictions
            signal.aiPrediction = predictions;
            signal.sentiment = sentiment;
            signal.strategy = strategyName;
            signal.timestamp = Date.now();
            
            // Calculate risk-adjusted confidence
            signal.confidence = this.calculateRiskAdjustedConfidence(signal, indicators);
            
            // Add expected return calculation
            signal.expectedReturn = this.calculateExpectedReturn(signal, predictions);
        }
        
        return signal;
    }
    
    // Trend Following Signal Generation
    generateTrendFollowingSignal(symbol, market, indicators, params) {
        const { fastMA, slowMA, rsi, macd } = params;
        
        // Moving Average Crossover
        const fastMAValue = indicators.ma[fastMA];
        const slowMAValue = indicators.ma[slowMA];
        const prevFastMA = indicators.ma[fastMA + '_prev'];
        const prevSlowMA = indicators.ma[slowMA + '_prev'];
        
        // RSI conditions
        const rsiValue = indicators.rsi[rsi.period];
        
        // MACD conditions
        const macdLine = indicators.macd.line;
        const macdSignal = indicators.macd.signal;
        const macdHist = indicators.macd.histogram;
        
        // Volume confirmation
        const volumeRatio = market.volume / indicators.avgVolume;
        
        let signal = null;
        let confidence = 0;
        
        // Bullish signal
        if (fastMAValue > slowMAValue && 
            prevFastMA <= prevSlowMA && // Crossover
            rsiValue > 30 && rsiValue < 70 && // Not overbought/oversold
            macdLine > macdSignal && // MACD bullish
            volumeRatio > params.volumeThreshold) {
            
            signal = {
                type: 'BUY',
                symbol: symbol,
                price: market.price,
                confidence: 0.7,
                strength: this.calculateTrendStrength(indicators)
            };
            
            // Adjust confidence based on trend strength
            confidence = 0.7 + (signal.strength * 0.3);
        }
        
        // Bearish signal
        else if (fastMAValue < slowMAValue && 
                 prevFastMA >= prevSlowMA && // Crossover
                 rsiValue > 30 && rsiValue < 70 &&
                 macdLine < macdSignal && // MACD bearish
                 volumeRatio > params.volumeThreshold) {
            
            signal = {
                type: 'SELL',
                symbol: symbol,
                price: market.price,
                confidence: 0.7,
                strength: this.calculateTrendStrength(indicators)
            };
            
            confidence = 0.7 + (signal.strength * 0.3);
        }
        
        if (signal) {
            signal.confidence = Math.min(confidence, 0.95);
            signal.stopLoss = this.calculateStopLoss(signal.type, signal.price);
            signal.takeProfit = this.calculateTakeProfit(signal.type, signal.price);
        }
        
        return signal;
    }
    
    // Mean Reversion Signal Generation
    generateMeanReversionSignal(symbol, market, indicators, params) {
        const bbUpper = indicators.bollingerBands.upper;
        const bbLower = indicators.bollingerBands.lower;
        const bbMiddle = indicators.bollingerBands.middle;
        const rsiValue = indicators.rsi[params.rsiPeriod];
        const zScore = (market.price - bbMiddle) / (bbUpper - bbMiddle);
        
        let signal = null;
        
        // Oversold condition (Buy signal)
        if (market.price <= bbLower && 
            rsiValue <= params.rsiOversold && 
            Math.abs(zScore) >= params.zScoreThreshold) {
            
            signal = {
                type: 'BUY',
                symbol: symbol,
                price: market.price,
                confidence: 0.75,
                target: bbMiddle,
                reason: 'OVERSOLD_REVERSION'
            };
        }
        
        // Overbought condition (Sell signal)
        else if (market.price >= bbUpper && 
                 rsiValue >= params.rsioverbought && 
                 Math.abs(zScore) >= params.zScoreThreshold) {
            
            signal = {
                type: 'SELL',
                symbol: symbol,
                price: market.price,
                confidence: 0.75,
                target: bbMiddle,
                reason: 'OVERBOUGHT_REVERSION'
            };
        }
        
        if (signal) {
            signal.stopLoss = this.calculateStopLoss(signal.type, signal.price);
            signal.takeProfit = signal.target;
        }
        
        return signal;
    }
    
    // Breakout Signal Generation
    generateBreakoutSignal(symbol, market, indicators, params) {
        const resistance = indicators.resistance;
        const support = indicators.support;
        const volumeRatio = market.volume / indicators.avgVolume;
        const consolidationDuration = indicators.consolidationPeriod;
        
        let signal = null;
        
        // Bullish breakout
        if (market.price > resistance && 
            volumeRatio >= params.volumeMultiplier &&
            consolidationDuration >= params.consolidationPeriod) {
            
            const breakoutStrength = (market.price - resistance) / resistance;
            
            if (breakoutStrength >= params.breakoutThreshold) {
                signal = {
                    type: 'BUY',
                    symbol: symbol,
                    price: market.price,
                    confidence: 0.8,
                    breakoutLevel: resistance,
                    strength: breakoutStrength,
                    reason: 'BULLISH_BREAKOUT'
                };
            }
        }
        
        // Bearish breakdown
        else if (market.price < support && 
                 volumeRatio >= params.volumeMultiplier &&
                 consolidationDuration >= params.consolidationPeriod) {
            
            const breakdownStrength = (support - market.price) / support;
            
            if (breakdownStrength >= params.breakoutThreshold) {
                signal = {
                    type: 'SELL',
                    symbol: symbol,
                    price: market.price,
                    confidence: 0.8,
                    breakoutLevel: support,
                    strength: breakdownStrength,
                    reason: 'BEARISH_BREAKDOWN'
                };
            }
        }
        
        if (signal) {
            signal.stopLoss = signal.type === 'BUY' ? 
                signal.breakoutLevel * (1 - this.config.stopLossPercent) :
                signal.breakoutLevel * (1 + this.config.stopLossPercent);
            
            signal.takeProfit = this.calculateTakeProfit(signal.type, signal.price);
        }
        
        return signal;
    }
    
    // DCA Signal Generation
    generateDCASignal(symbol, market, sentiment, params) {
        // Only buy signals for DCA
        const priceDropFromATH = this.calculatePriceDropFromATH(symbol);
        const sentimentScore = sentiment.overall.score;
        const volatility = this.calculateVolatility(symbol);
        
        let signal = null;
        
        // DCA buy conditions
        if (priceDropFromATH >= params.priceDropThreshold ||
            (sentimentScore < 40 && params.sentimentFilter)) { // Buy on fear
            
            let amount = params.investmentAmount;
            
            // Adjust amount based on volatility
            if (params.volatilityAdjustment) {
                amount *= (1 + volatility); // Buy more when volatile
            }
            
            signal = {
                type: 'BUY',
                symbol: symbol,
                price: market.price,
                amount: amount,
                confidence: 0.6,
                reason: 'DCA_ACCUMULATION',
                dcaLevel: priceDropFromATH
            };
        }
        
        return signal;
    }
    
    // Scalping Signal Generation
    generateScalpingSignal(symbol, market, indicators, params) {
        // High-frequency signals for scalping
        const shortMA = indicators.ma[5];  // 5-period MA
        const microTrend = market.price > shortMA ? 'UP' : 'DOWN';
        const priceVelocity = this.calculatePriceVelocity(symbol);
        const spreadCost = this.calculateSpreadCost(symbol);
        
        let signal = null;
        
        // Quick momentum scalp
        if (Math.abs(priceVelocity) > 0.001 && // Sufficient momentum
            spreadCost < params.targetProfit / 2) { // Spread doesn't eat profit
            
            signal = {
                type: priceVelocity > 0 ? 'BUY' : 'SELL',
                symbol: symbol,
                price: market.price,
                confidence: 0.6,
                holdTime: params.maxHoldTime,
                reason: 'MOMENTUM_SCALP'
            };
            
            signal.stopLoss = signal.type === 'BUY' ?
                signal.price * (1 - params.stopLoss) :
                signal.price * (1 + params.stopLoss);
                
            signal.takeProfit = signal.type === 'BUY' ?
                signal.price * (1 + params.targetProfit) :
                signal.price * (1 - params.targetProfit);
        }
        
        return signal;
    }
    
    // Process signals and execute trades
    async processSignals(signals) {
        if (signals.length === 0) return;
        
        console.log(`🔄 Processing ${signals.length} trading signals...`);
        
        for (const signal of signals) {
            try {
                // Risk management checks
                if (!await this.passesRiskChecks(signal)) {
                    console.log(`⚠️ Signal ${signal.symbol} failed risk checks`);
                    continue;
                }
                
                // Portfolio allocation checks
                if (!this.hasAvailableCapital(signal)) {
                    console.log(`💰 Insufficient capital for ${signal.symbol} signal`);
                    continue;
                }
                
                // Position size calculation
                const positionSize = this.calculatePositionSize(signal);
                if (positionSize <= 0) continue;
                
                // Execute the trade
                const executionResult = await this.executeTrade(signal, positionSize);
                
                if (executionResult.success) {
                    console.log(`✅ Trade executed: ${signal.type} ${signal.symbol} at ${signal.price}`);
                    
                    // Record the trade
                    this.recordTrade(signal, executionResult);
                    
                    // Update strategy performance
                    this.updateStrategyPerformance(signal.strategy, executionResult);
                    
                } else {
                    console.log(`❌ Trade execution failed: ${executionResult.error}`);
                }
                
            } catch (error) {
                console.error(`❌ Error processing signal for ${signal.symbol}:`, error);
            }
        }
    }
    
    // Execute a trade
    async executeTrade(signal, positionSize) {
        try {
            const orderData = {
                symbol: signal.symbol,
                type: signal.type,
                amount: positionSize,
                price: signal.price,
                stopLoss: signal.stopLoss,
                takeProfit: signal.takeProfit,
                strategy: signal.strategy,
                confidence: signal.confidence,
                timestamp: Date.now()
            };
            
            // Simulate order execution (in real implementation, this would call exchange API)
            const executionResult = await this.simulateOrderExecution(orderData);
            
            if (executionResult.success) {
                // Add to positions tracking
                this.positions.set(executionResult.orderId, {
                    ...orderData,
                    orderId: executionResult.orderId,
                    executedPrice: executionResult.executedPrice,
                    status: 'OPEN',
                    pnl: 0,
                    openTime: Date.now()
                });
                
                // Set stop loss and take profit orders
                if (orderData.stopLoss) {
                    await this.placeStopLossOrder(executionResult.orderId, orderData);
                }
                
                if (orderData.takeProfit) {
                    await this.placeTakeProfitOrder(executionResult.orderId, orderData);
                }
            }
            
            return executionResult;
            
        } catch (error) {
            return { success: false, error: error.message };
        }
    }
    
    // Simulate order execution
    async simulateOrderExecution(orderData) {
        // Simulate network delay
        await new Promise(resolve => setTimeout(resolve, Math.random() * 1000 + 100));
        
        // Simulate slippage
        const slippage = (Math.random() - 0.5) * 0.001; // ±0.1% slippage
        const executedPrice = orderData.price * (1 + slippage);
        
        // Simulate execution success/failure (95% success rate)
        const success = Math.random() > 0.05;
        
        if (success) {
            const orderId = 'order_' + Date.now() + '_' + Math.random().toString(36).substr(2, 6);
            
            return {
                success: true,
                orderId: orderId,
                executedPrice: executedPrice,
                executedAmount: orderData.amount,
                fee: orderData.amount * executedPrice * 0.001, // 0.1% fee
                timestamp: Date.now()
            };
        } else {
            return {
                success: false,
                error: 'Order execution failed - insufficient liquidity'
            };
        }
    }
    
    // Calculate position size based on risk management
    calculatePositionSize(signal) {
        const accountBalance = this.platform.portfolio.balance || 1000;
        const riskAmount = accountBalance * this.config.maxRiskPerTrade;
        
        // Calculate risk per share
        const riskPerShare = Math.abs(signal.price - signal.stopLoss);
        
        if (riskPerShare <= 0) return 0;
        
        // Position size = Risk Amount / Risk Per Share
        let positionSize = riskAmount / riskPerShare;
        
        // Apply maximum position size limits
        const maxPositionValue = accountBalance * 0.1; // Max 10% of account per position
        const maxPositionSize = maxPositionValue / signal.price;
        
        positionSize = Math.min(positionSize, maxPositionSize);
        
        // Ensure minimum position size
        const minPositionValue = 10; // $10 minimum
        const minPositionSize = minPositionValue / signal.price;
        
        if (positionSize < minPositionSize) return 0;
        
        return Math.floor(positionSize * 1000000) / 1000000; // Round to 6 decimal places
    }
    
    // Risk management checks
    async passesRiskChecks(signal) {
        // Check maximum open positions
        if (this.positions.size >= this.config.maxOpenPositions) {
            return false;
        }
        
        // Check daily loss limit
        const todayLoss = this.calculateTodayLoss();
        const accountBalance = this.platform.portfolio.balance || 1000;
        
        if (todayLoss >= accountBalance * this.config.maxDailyLoss) {
            return false;
        }
        
        // Check if already have position in this symbol
        const existingPosition = Array.from(this.positions.values())
            .find(pos => pos.symbol === signal.symbol && pos.status === 'OPEN');
        
        if (existingPosition) {
            return false; // Don't add to existing position
        }
        
        // Risk assessment by AI model
        const riskAssessment = await this.aiModels.riskAssessor.assessRisk(signal);
        
        if (riskAssessment.level === 'extreme') {
            return false;
        }
        
        return true;
    }
    
    // Update positions and check for exit conditions
    async updatePositions() {
        for (const [orderId, position] of this.positions) {
            try {
                if (position.status !== 'OPEN') continue;
                
                // Get current market price
                const currentMarket = this.platform.marketData.get(position.symbol);
                if (!currentMarket) continue;
                
                const currentPrice = currentMarket.price;
                
                // Calculate current PnL
                const pnl = position.type === 'BUY' ?
                    (currentPrice - position.executedPrice) * position.amount :
                    (position.executedPrice - currentPrice) * position.amount;
                
                position.pnl = pnl;
                position.pnlPercent = (pnl / (position.executedPrice * position.amount)) * 100;
                
                // Check exit conditions
                await this.checkExitConditions(orderId, position, currentPrice);
                
            } catch (error) {
                console.error(`❌ Error updating position ${orderId}:`, error);
            }
        }
    }
    
    // Check exit conditions for positions
    async checkExitConditions(orderId, position, currentPrice) {
        let shouldExit = false;
        let exitReason = '';
        
        // Stop loss check
        if (position.stopLoss) {
            if ((position.type === 'BUY' && currentPrice <= position.stopLoss) ||
                (position.type === 'SELL' && currentPrice >= position.stopLoss)) {
                shouldExit = true;
                exitReason = 'STOP_LOSS';
            }
        }
        
        // Take profit check
        if (position.takeProfit && !shouldExit) {
            if ((position.type === 'BUY' && currentPrice >= position.takeProfit) ||
                (position.type === 'SELL' && currentPrice <= position.takeProfit)) {
                shouldExit = true;
                exitReason = 'TAKE_PROFIT';
            }
        }
        
        // Trailing stop check
        if (!shouldExit && this.config.trailingStopPercent > 0) {
            const trailingStop = this.calculateTrailingStop(position, currentPrice);
            if (trailingStop.shouldExit) {
                shouldExit = true;
                exitReason = 'TRAILING_STOP';
                position.stopLoss = trailingStop.newStopLoss;
            }
        }
        
        // Time-based exit (for scalping)
        if (position.holdTime && !shouldExit) {
            const holdDuration = Date.now() - position.openTime;
            if (holdDuration >= position.holdTime) {
                shouldExit = true;
                exitReason = 'TIME_EXIT';
            }
        }
        
        // AI-based exit signal
        if (!shouldExit) {
            const exitSignal = await this.generateExitSignal(position, currentPrice);
            if (exitSignal.shouldExit) {
                shouldExit = true;
                exitReason = 'AI_EXIT';
            }
        }
        
        // Execute exit if needed
        if (shouldExit) {
            await this.exitPosition(orderId, position, currentPrice, exitReason);
        }
    }
    
    // Exit a position
    async exitPosition(orderId, position, exitPrice, reason) {
        try {
            // Simulate closing the position
            const closeResult = await this.simulateOrderExecution({
                symbol: position.symbol,
                type: position.type === 'BUY' ? 'SELL' : 'BUY',
                amount: position.amount,
                price: exitPrice
            });
            
            if (closeResult.success) {
                // Calculate final PnL
                const finalPnL = position.type === 'BUY' ?
                    (closeResult.executedPrice - position.executedPrice) * position.amount :
                    (position.executedPrice - closeResult.executedPrice) * position.amount;
                
                // Update position
                position.status = 'CLOSED';
                position.closePrice = closeResult.executedPrice;
                position.closeTime = Date.now();
                position.finalPnL = finalPnL - closeResult.fee;
                position.exitReason = reason;
                
                // Record the trade in history
                this.tradingHistory.push({...position});
                
                // Remove from active positions
                this.positions.delete(orderId);
                
                // Update performance tracking
                this.performanceTracker.recordTrade(position);
                
                // Update account balance
                this.platform.portfolio.balance += position.finalPnL;
                
                console.log(`📈 Position closed: ${position.symbol} ${reason} PnL: ${finalPnL.toFixed(4)}`);
                
                // Show notification for significant trades
                if (Math.abs(finalPnL) > 10) {
                    const pnlText = finalPnL > 0 ? `+$${finalPnL.toFixed(2)}` : `-$${Math.abs(finalPnL).toFixed(2)}`;
                    this.platform.showNotification(
                        `🤖 ${position.symbol} position closed: ${pnlText} (${reason})`,
                        finalPnL > 0 ? 'success' : 'error'
                    );
                }
            }
            
        } catch (error) {
            console.error(`❌ Error closing position ${orderId}:`, error);
        }
    }
    
    // Performance tracking and analysis
    getBotPerformance() {
        const totalTrades = this.tradingHistory.length;
        const winningTrades = this.tradingHistory.filter(trade => trade.finalPnL > 0);
        const losingTrades = this.tradingHistory.filter(trade => trade.finalPnL < 0);
        
        const totalPnL = this.tradingHistory.reduce((sum, trade) => sum + trade.finalPnL, 0);
        const winRate = totalTrades > 0 ? (winningTrades.length / totalTrades) * 100 : 0;
        
        const avgWin = winningTrades.length > 0 ? 
            winningTrades.reduce((sum, trade) => sum + trade.finalPnL, 0) / winningTrades.length : 0;
        
        const avgLoss = losingTrades.length > 0 ? 
            Math.abs(losingTrades.reduce((sum, trade) => sum + trade.finalPnL, 0) / losingTrades.length) : 0;
        
        const profitFactor = avgLoss > 0 ? (avgWin * winningTrades.length) / (avgLoss * losingTrades.length) : 0;
        
        return {
            isActive: this.isActive,
            totalTrades,
            winningTrades: winningTrades.length,
            losingTrades: losingTrades.length,
            winRate: winRate.toFixed(2),
            totalPnL: totalPnL.toFixed(4),
            avgWin: avgWin.toFixed(4),
            avgLoss: avgLoss.toFixed(4),
            profitFactor: profitFactor.toFixed(2),
            activePositions: this.positions.size,
            strategies: Object.fromEntries(this.strategies),
            config: this.config
        };
    }
    
    // Get bot status for UI
    getBotStatus() {
        return {
            isActive: this.isActive,
            activePositions: this.positions.size,
            todayTrades: this.getTodayTradesCount(),
            todayPnL: this.getTodayPnL(),
            alerts: this.alerts.slice(-5), // Last 5 alerts
            lastSignalTime: this.getLastSignalTime(),
            performance: this.getBotPerformance()
        };
    }
    
    // Utility methods
    calculateTodayLoss() {
        const today = new Date().toDateString();
        return this.tradingHistory
            .filter(trade => new Date(trade.closeTime).toDateString() === today && trade.finalPnL < 0)
            .reduce((sum, trade) => sum + Math.abs(trade.finalPnL), 0);
    }
    
    getTodayTradesCount() {
        const today = new Date().toDateString();
        return this.tradingHistory
            .filter(trade => new Date(trade.closeTime).toDateString() === today).length;
    }
    
    getTodayPnL() {
        const today = new Date().toDateString();
        return this.tradingHistory
            .filter(trade => new Date(trade.closeTime).toDateString() === today)
            .reduce((sum, trade) => sum + trade.finalPnL, 0);
    }
    
    getLastSignalTime() {
        return this.tradingHistory.length > 0 ? 
            Math.max(...this.tradingHistory.map(trade => trade.openTime)) : 0;
    }
    
    // Error handling
    handleTradingError(error) {
        console.error('❌ Trading error:', error);
        
        this.alerts.push({
            type: 'ERROR',
            message: error.message,
            timestamp: Date.now()
        });
        
        // Emergency stop on critical errors
        if (this.isCriticalError(error)) {
            this.emergencyStop();
        }
    }
    
    emergencyStop() {
        console.log('🚨 Emergency stop triggered!');
        this.isActive = false;
        this.platform.showNotification('🚨 Trading bot emergency stop activated!', 'error');
    }
    
    // Additional utility methods would go here...
    // (calculateIndicators, analyzeSentiment, predictPrice, etc.)
}

// Risk Manager Class
class RiskManager {
    constructor() {
        this.riskLimits = {
            maxDailyLoss: 0.05,
            maxDrawdown: 0.15,
            maxPositionSize: 0.1,
            maxCorrelation: 0.7
        };
    }
    
    assessPositionRisk(position, marketData) {
        // Risk assessment logic
        return {
            riskLevel: 'MEDIUM',
            riskScore: 0.6,
            recommendations: []
        };
    }
}

// Portfolio Manager Class
class PortfolioManager {
    constructor() {
        this.allocations = new Map();
        this.rebalanceThreshold = 0.05;
    }
    
    rebalancePortfolio(targetAllocations) {
        // Portfolio rebalancing logic
        console.log('⚖️ Rebalancing portfolio...');
    }
}

// Signal Generator Class
class SignalGenerator {
    constructor() {
        this.indicators = new Map();
        this.patterns = new Map();
    }
    
    generateSignals(marketData) {
        // Signal generation logic
        return [];
    }
}

// Execution Engine Class
class ExecutionEngine {
    constructor(platform) {
        this.platform = platform;
        this.orderQueue = [];
    }
    
    executeOrder(orderData) {
        // Order execution logic
        return { success: true, orderId: 'order_123' };
    }
}

// Performance Tracker Class
class PerformanceTracker {
    constructor() {
        this.metrics = new Map();
        this.benchmarks = new Map();
    }
    
    recordTrade(trade) {
        // Performance tracking logic
        console.log('📊 Recording trade performance...');
    }
}

// Machine Learning Predictor Class
class MLPredictor {
    constructor() {
        this.models = new Map();
        this.features = [];
    }
    
    predictPrice(symbol, features) {
        // ML prediction logic
        return {
            direction: 'UP',
            confidence: 0.7,
            target: 0.026
        };
    }
}

// AI Model Classes
class PricePredictionModel {
    initialize(config) {
        console.log('🧠 Initializing Price Prediction Model...');
        this.config = config;
    }
    
    predict(features) {
        // Price prediction logic
        return {
            '1h': { price: 0.025500, confidence: 0.7 },
            '4h': { price: 0.026000, confidence: 0.65 },
            '1d': { price: 0.027000, confidence: 0.6 }
        };
    }
}

class SentimentAnalysisModel {
    initialize(config) {
        console.log('🧠 Initializing Sentiment Analysis Model...');
        this.config = config;
    }
    
    analyzeSentiment(text) {
        // Sentiment analysis logic
        return {
            score: 0.7,
            label: 'BULLISH',
            confidence: 0.8
        };
    }
}

class RiskAssessmentModel {
    initialize(config) {
        console.log('🧠 Initializing Risk Assessment Model...');
        this.config = config;
    }
    
    assessRisk(signal) {
        // Risk assessment logic
        return {
            level: 'MEDIUM',
            score: 0.6,
            factors: ['volatility', 'liquidity']
        };
    }
}

class PatternRecognitionModel {
    initialize(config) {
        console.log('🧠 Initializing Pattern Recognition Model...');
        this.config = config;
    }
    
    recognizePatterns(priceData) {
        // Pattern recognition logic
        return {
            patterns: ['DOUBLE_BOTTOM', 'BULLISH_FLAG'],
            confidence: 0.75
        };
    }
}

class VolatilityPredictionModel {
    initialize(config) {
        console.log('🧠 Initializing Volatility Prediction Model...');
        this.config = config;
    }
    
    predictVolatility(marketData) {
        // Volatility prediction logic
        return {
            current: 0.3,
            predicted: 0.25,
            trend: 'DECREASING'
        };
    }
}

// Export for integration
if (typeof window !== 'undefined') {
    window.AITradingBot = AITradingBot;
}

// Integration with existing platform
document.addEventListener('DOMContentLoaded', () => {
    if (window.tradingPlatform) {
        window.aiTradingBot = new AITradingBot(window.tradingPlatform);
        console.log('🤖 AI Trading Bot integrated with YouNEXT platform!');
    }
});

