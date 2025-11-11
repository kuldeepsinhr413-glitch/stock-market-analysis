# stock-market-analysis
import React, { useState, useEffect } from 'react';
import { LineChart, Line, BarChart, Bar, XAxis, YAxis, CartesianGrid, Tooltip, Legend, ResponsiveContainer, Area, AreaChart } from 'recharts';
import { TrendingUp, TrendingDown, DollarSign, Activity, BarChart3, Calendar } from 'lucide-react';

const StockMarketAnalysis = () => {
  const [stockData, setStockData] = useState([]);
  const [selectedStock, setSelectedStock] = useState('AAPL');
  const [timeRange, setTimeRange] = useState('1M');
  const [metrics, setMetrics] = useState({});
  const [loading, setLoading] = useState(false);

  // Sample stock symbols
  const stocks = [
    { symbol: 'AAPL', name: 'Apple Inc.' },
    { symbol: 'GOOGL', name: 'Alphabet Inc.' },
    { symbol: 'MSFT', name: 'Microsoft Corp.' },
    { symbol: 'AMZN', name: 'Amazon.com Inc.' },
    { symbol: 'TSLA', name: 'Tesla Inc.' }
  ];

  // Generate sample data (In production, this would fetch from Yahoo Finance API)
  const generateSampleData = (symbol, range) => {
    const days = range === '1M' ? 30 : range === '3M' ? 90 : range === '6M' ? 180 : 365;
    const basePrice = { 'AAPL': 150, 'GOOGL': 2800, 'MSFT': 300, 'AMZN': 3200, 'TSLA': 800 }[symbol];
    const data = [];
    
    let price = basePrice;
    const today = new Date();
    
    for (let i = days; i >= 0; i--) {
      const date = new Date(today);
      date.setDate(date.getDate() - i);
      
      // Simulate price movement
      const change = (Math.random() - 0.48) * (basePrice * 0.02);
      price = Math.max(price + change, basePrice * 0.7);
      
      const open = price + (Math.random() - 0.5) * 2;
      const close = price + (Math.random() - 0.5) * 2;
      const high = Math.max(open, close) + Math.random() * 3;
      const low = Math.min(open, close) - Math.random() * 3;
      const volume = Math.floor(Math.random() * 100000000) + 50000000;
      
      data.push({
        date: date.toISOString().split('T')[0],
        open: parseFloat(open.toFixed(2)),
        high: parseFloat(high.toFixed(2)),
        low: parseFloat(low.toFixed(2)),
        close: parseFloat(close.toFixed(2)),
        volume: volume,
        change: parseFloat((close - open).toFixed(2))
      });
    }
    
    return data;
  };

  // Calculate technical indicators and metrics
  const calculateMetrics = (data) => {
    if (data.length === 0) return {};

    const closes = data.map(d => d.close);
    const volumes = data.map(d => d.volume);
    
    // Moving averages
    const sma20 = closes.slice(-20).reduce((a, b) => a + b, 0) / Math.min(20, closes.length);
    const sma50 = closes.slice(-50).reduce((a, b) => a + b, 0) / Math.min(50, closes.length);
    
    // Price change
    const currentPrice = closes[closes.length - 1];
    const previousPrice = closes[0];
    const priceChange = currentPrice - previousPrice;
    const priceChangePercent = (priceChange / previousPrice) * 100;
    
    // Volatility (standard deviation)
    const mean = closes.reduce((a, b) => a + b, 0) / closes.length;
    const variance = closes.reduce((acc, val) => acc + Math.pow(val - mean, 2), 0) / closes.length;
    const volatility = Math.sqrt(variance);
    
    // Trading volume
    const avgVolume = volumes.reduce((a, b) => a + b, 0) / volumes.length;
    const currentVolume = volumes[volumes.length - 1];
    
    // High and Low
    const high52Week = Math.max(...closes);
    const low52Week = Math.min(...closes);
    
    return {
      currentPrice: currentPrice.toFixed(2),
      priceChange: priceChange.toFixed(2),
      priceChangePercent: priceChangePercent.toFixed(2),
      sma20: sma20.toFixed(2),
      sma50: sma50.toFixed(2),
      volatility: volatility.toFixed(2),
      avgVolume: (avgVolume / 1000000).toFixed(2),
      currentVolume: (currentVolume / 1000000).toFixed(2),
      high52Week: high52Week.toFixed(2),
      low52Week: low52Week.toFixed(2)
    };
  };

  // Load data when stock or time range changes
  useEffect(() => {
    setLoading(true);
    setTimeout(() => {
      const data = generateSampleData(selectedStock, timeRange);
      setStockData(data);
      setMetrics(calculateMetrics(data));
      setLoading(false);
    }, 500);
  }, [selectedStock, timeRange]);

  // Add moving averages to chart data
  const chartData = stockData.map((item, index) => {
    const closes = stockData.slice(0, index + 1).map(d => d.close);
    const sma20 = closes.slice(-20).reduce((a, b) => a + b, 0) / Math.min(20, closes.length);
    const sma50 = closes.slice(-50).reduce((a, b) => a + b, 0) / Math.min(50, closes.length);
    
    return {
      ...item,
      sma20: parseFloat(sma20.toFixed(2)),
      sma50: parseFloat(sma50.toFixed(2))
    };
  });

  const MetricCard = ({ title, value, subValue, icon: Icon, trend }) => (
    <div className="bg-white rounded-lg shadow p-4 border border-gray-200">
      <div className="flex items-center justify-between mb-2">
        <span className="text-sm font-medium text-gray-600">{title}</span>
        <Icon className="w-5 h-5 text-gray-400" />
      </div>
      <div className="text-2xl font-bold text-gray-900">{value}</div>
      {subValue && (
        <div className={text-sm mt-1 flex items-center ${trend === 'up' ? 'text-green-600' : trend === 'down' ? 'text-red-600' : 'text-gray-600'}}>
          {trend === 'up' && <TrendingUp className="w-4 h-4 mr-1" />}
          {trend === 'down' && <TrendingDown className="w-4 h-4 mr-1" />}
          {subValue}
        </div>
      )}
    </div>
  );

  return (
    <div className="min-h-screen bg-gray-50 p-6">
      <div className="max-w-7xl mx-auto">
        {/* Header */}
        <div className="mb-6">
          <h1 className="text-3xl font-bold text-gray-900 mb-2">Stock Market Analysis Dashboard</h1>
          <p className="text-gray-600">Historical stock market data analysis with technical indicators</p>
        </div>

        {/* Controls */}
        <div className="bg-white rounded-lg shadow p-6 mb-6">
          <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
            <div>
              <label className="block text-sm font-medium text-gray-700 mb-2">Select Stock</label>
              <select
                value={selectedStock}
                onChange={(e) => setSelectedStock(e.target.value)}
                className="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
              >
                {stocks.map(stock => (
                  <option key={stock.symbol} value={stock.symbol}>
                    {stock.symbol} - {stock.name}
                  </option>
                ))}
              </select>
            </div>
            <div>
              <label className="block text-sm font-medium text-gray-700 mb-2">Time Range</label>
              <div className="flex gap-2">
                {['1M', '3M', '6M', '1Y'].map(range => (
                  <button
                    key={range}
                    onClick={() => setTimeRange(range)}
                    className={`flex-1 px-4 py-2 rounded-lg font-medium transition-colors ${
                      timeRange === range
                        ? 'bg-blue-600 text-white'
                        : 'bg-gray-100 text-gray-700 hover:bg-gray-200'
                    }`}
                  >
                    {range}
                  </button>
                ))}
              </div>
            </div>
          </div>
        </div>

        {/* Metrics Cards */}
        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4 mb-6">
          <MetricCard
            title="Current Price"
            value={$${metrics.currentPrice || '0.00'}}
            subValue={${metrics.priceChange > 0 ? '+' : ''}${metrics.priceChange} (${metrics.priceChangePercent}%)}
            icon={DollarSign}
            trend={metrics.priceChange > 0 ? 'up' : 'down'}
          />
          <MetricCard
            title="Volatility (Std Dev)"
            value={$${metrics.volatility || '0.00'}}
            icon={Activity}
          />
          <MetricCard
            title="Avg Volume"
            value={${metrics.avgVolume || '0.00'}M}
            subValue={Current: ${metrics.currentVolume}M}
            icon={BarChart3}
          />
          <MetricCard
            title="52W High / Low"
            value={$${metrics.high52Week || '0.00'}}
            subValue={Low: $${metrics.low52Week || '0.00'}}
            icon={Calendar}
          />
        </div>

        {/* Price Chart */}
        <div className="bg-white rounded-lg shadow p-6 mb-6">
          <h2 className="text-xl font-semibold text-gray-900 mb-4">Price History & Moving Averages</h2>
          {loading ? (
            <div className="h-96 flex items-center justify-center">
              <div className="text-gray-500">Loading data...</div>
            </div>
          ) : (
            <ResponsiveContainer width="100%" height={400}>
              <LineChart data={chartData}>
                <CartesianGrid strokeDasharray="3 3" />
                <XAxis dataKey="date" tick={{ fontSize: 12 }} />
                <YAxis domain={['auto', 'auto']} tick={{ fontSize: 12 }} />
                <Tooltip />
                <Legend />
                <Line type="monotone" dataKey="close" stroke="#2563eb" strokeWidth={2} name="Close Price" dot={false} />
                <Line type="monotone" dataKey="sma20" stroke="#10b981" strokeWidth={1.5} name="SMA 20" dot={false} strokeDasharray="5 5" />
                <Line type="monotone" dataKey="sma50" stroke="#f59e0b" strokeWidth={1.5} name="SMA 50" dot={false} strokeDasharray="5 5" />
              </LineChart>
            </ResponsiveContainer>
          )}
        </div>

        {/* Volume Chart */}
        <div className="bg-white rounded-lg shadow p-6 mb-6">
          <h2 className="text-xl font-semibold text-gray-900 mb-4">Trading Volume</h2>
          {loading ? (
            <div className="h-64 flex items-center justify-center">
              <div className="text-gray-500">Loading data...</div>
            </div>
          ) : (
            <ResponsiveContainer width="100%" height={250}>
              <BarChart data={chartData}>
                <CartesianGrid strokeDasharray="3 3" />
                <XAxis dataKey="date" tick={{ fontSize: 12 }} />
                <YAxis tick={{ fontSize: 12 }} />
                <Tooltip />
                <Bar dataKey="volume" fill="#6366f1" name="Volume" />
              </BarChart>
            </ResponsiveContainer>
          )}
        </div>

        {/* Price Range Chart */}
        <div className="bg-white rounded-lg shadow p-6">
          <h2 className="text-xl font-semibold text-gray-900 mb-4">Daily Price Range (High-Low)</h2>
          {loading ? (
            <div className="h-64 flex items-center justify-center">
              <div className="text-gray-500">Loading data...</div>
            </div>
          ) : (
            <ResponsiveContainer width="100%" height={250}>
              <AreaChart data={chartData}>
                <CartesianGrid strokeDasharray="3 3" />
                <XAxis dataKey="date" tick={{ fontSize: 12 }} />
                <YAxis domain={['auto', 'auto']} tick={{ fontSize: 12 }} />
                <Tooltip />
                <Area type="monotone" dataKey="high" stackId="1" stroke="#ef4444" fill="#fca5a5" name="High" />
                <Area type="monotone" dataKey="low" stackId="2" stroke="#3b82f6" fill="#93c5fd" name="Low" />
              </AreaChart>
            </ResponsiveContainer>
          )}
        </div>

        {/* Technical Analysis Summary */}
        <div className="bg-white rounded-lg shadow p-6 mt-6">
          <h2 className="text-xl font-semibold text-gray-900 mb-4">Technical Analysis Summary</h2>
          <div className="space-y-3">
            <div className="flex justify-between items-center p-3 bg-gray-50 rounded">
              <span className="font-medium text-gray-700">20-Day Moving Average:</span>
              <span className="text-gray-900">${metrics.sma20}</span>
            </div>
            <div className="flex justify-between items-center p-3 bg-gray-50 rounded">
              <span className="font-medium text-gray-700">50-Day Moving Average:</span>
              <span className="text-gray-900">${metrics.sma50}</span>
            </div>
            <div className="flex justify-between items-center p-3 bg-gray-50 rounded">
              <span className="font-medium text-gray-700">Trend Indicator:</span>
              <span className={font-semibold ${parseFloat(metrics.sma20) > parseFloat(metrics.sma50) ? 'text-green-600' : 'text-red-600'}}>
                {parseFloat(metrics.sma20) > parseFloat(metrics.sma50) ? 'Bullish (SMA20 > SMA50)' : 'Bearish (SMA20 < SMA50)'}
              </span>
            </div>
          </div>
        </div>
      </div>
    </div>
  );
};

export default StockMarketAnalysis;
