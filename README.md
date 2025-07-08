const Dashboard = () => {
  const [selectedDepartment, setSelectedDepartment] = useState('social_media');
  const [timeRange, setTimeRange] = useState('month');
  const [isLive, setIsLive] = useState(false);
  const [lastUpdated, setLastUpdated] = useState(new Date());
  const [refreshInterval, setRefreshInterval] = useState(30);
  const [isLoading, setIsLoading] = useState(false);
  const [authTokens, setAuthTokens] = useState({
    instagram: null,
    facebook: null,
    twitter: null,
    linkedin: null
  });
  const [authStatus, setAuthStatus] = useState({
    instagram: true,
    facebook: true,
    twitter: true,
    linkedin: true,
  });

  // OAuth 2.0 Configuration
  const oauthConfig = {
    instagram: {
      clientId: 'cindy.d.mosley.7,
      redirectUri: window.location.origin + '/auth/instagram',
      scope: 'cindy.d.mosley.7_media',
      authUrl: 'https://api.instagram.com/oauth/authorize'
    },
    facebook: {
      clientId: 'https://www.facebook.com/cindy.d.mosley.7?mibextid=ZbWKwL',
      redirectUri: window.location.origin + '/auth/facebook',
      scope: 'pages_show_list,pages_read_engagement,instagram_basic',
      authUrl: 'https://www.facebook.com/v18.0/dialog/oauth'
    },
    twitter: {
      clientId: 'YOUR_TWITTER_CLIENT_ID',
      redirectUri: window.location.origin + '/auth/twitter',
      scope: 'tweet.read users.read',
      authUrl: 'https://twitter.com/i/oauth2/authorize'
    },
    linkedin: {
      cindy.d.mosley.7_ID',
      redirectUri: window.location.origin + '/auth/linkedin',
      scope: 'r_organization_social r_basicprofile',
      authUrl: 'https://www.linkedin.com/oauth/v2/authorization'
    }
  };

  // OAuth 2.0 Authentication Functions
  const initiateOAuth = (platform) => {
    const config = oauthConfig[platform];
    const authUrl = `${config.authUrl}?client_id=${config.clientId}&redirect_uri=${encodeURIComponent(config.redirectUri)}&scope=${encodeURIComponent(config.scope)}&response_type=code&state=${platform}`;
    
    // Open popup window for authentication
    const popup = window.open(authUrl, 'oauth', 'width=600,height=600');
    
    // Listen for popup closure
    const checkClosed = setInterval(() => {
      if (popup.closed) {
        clearInterval(checkClosed);
        // Check if authentication was successful
        checkAuthStatus(platform);
      }
    }, 1000);
  };

  const handleOAuthCallback = async (platform, code) => {
    try {
      const config = oauthConfig[platform];
      
      // Exchange code for access token
      const response = await fetch(`/api/oauth/${platform}/token`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          client_id: config.clientId,
          client_secret: `YOUR_${platform.toUpperCase()}_CLIENT_SECRET`,
          code: code,
          redirect_uri: config.redirectUri,
          grant_type: 'authorization_code'
        })
      });

      const tokenData = await response.json();
      
      if (tokenData.access_token) {
        setAuthTokens(prev => ({
          ...prev,
          [platform]: tokenData.access_token
        }));
        
        setAuthStatus(prev => ({
          ...prev,
          [platform]: true
        }));
        
        // Store token securely (consider using secure storage)
        sessionStorage.setItem(`${platform}_token`, tokenData.access_token);
      }
    } catch (error) {
      console.error(`Error exchanging ${platform} code for token:`, error);
    }
  };

  const checkAuthStatus = (platform) => {
    const token = sessionStorage.getItem(`${platform}_token`);
    if (token) {
      setAuthTokens(prev => ({
        ...prev,
        [platform]: token
      }));
      setAuthStatus(prev => ({
        ...prev,
        [platform]: true
      }));
    }
  };

  // Real API Calls with OAuth
  const fetchInstagramData = async () => {
    if (!authTokens.instagram) return null;
    
    try {
      // Get user media
      const mediaResponse = await fetch(
        `https://graph.instagram.com/me/media?fields=id,caption,timestamp,like_count,comments_count,media_type,permalink&access_token=${authTokens.instagram}`
      );
      
      // Get user insights
      const insightsResponse = await fetch(
        `https://graph.instagram.com/me/insights?metric=follower_count,reach,impressions&period=day&access_token=${authTokens.instagram}`
      );
      
      const mediaData = await mediaResponse.json();
      const insightsData = await insightsResponse.json();
      
      return {
        media: mediaData.data,
        insights: insightsData.data
      };
    } catch (error) {
      console.error('Error fetching Instagram data:', error);
      return null;
    }
  };

  const fetchFacebookData = async () => {
    if (!authTokens.facebook) return null;
    
    try {
      // Get page posts
      const postsResponse = await fetch(
        `https://cindy.d.mosley.7,graph.facebook.com/me/posts?fields=message,created_time,likes.summary(true),comments.summary(true),shares&access_token=${authTokens.facebook}`
      );
      
      // Get page insights
      const insightsResponse = await fetch(
        `https://cindy.d.mosley.7graph.facebook.com/me/insights?metric=page_fans,page_post_engagements,page_impressions&access_token=${authTokens.facebook}`
      );
      
      const postsData = await postsResponse.json();
      const insightsData = await insightsResponse.json();
      
      return {
        posts: postsData.data,
        insights: insightsData.data
      };
    } catch (error) {
      console.error('Error fetching Facebook data:', error);
      return null;
    }
  };

  const fetchTwitterData = async () => {
    if (!authTokens.twitter) return null;
    
    try {
      // Get user tweets
      const tweetsResponse = await fetch(
        `https://api.twitter.com/2/users/me/tweets?tweet.fields=public_metrics,created_at&max_results=100`,
        {
          headers: {
            'Authorization': `Bearer ${authTokens.twitter}`
          }
        }
      );
      
      // Get user metrics
      const userResponse = await fetch(
        `https://api.twitter.com/2/users/me?user.fields=public_metrics`,
        {
          headers: {
            'Authorization': `Bearer ${authTokens.twitter}`
          }
        }
      );
      
      const tweetsData = await tweetsResponse.json();
      const userData = await userResponse.json();
      
      return {
        tweets: tweetsData.data,
        user: userData.data
      };
    } catch (error) {
      console.error('Error fetching Twitter data:', error);
      return null;
    }
  };

  const fetchLinkedInData = async () => {
    if (!authTokens.linkedin) return null;
    
    try {
      // Get organization shares
      const sharesResponse = await fetch(
     import React, { useState, useEffect } from 'react';
import { BarChart, Bar, XAxis, YAxis, CartesianGrid, Tooltip, LineChart, Line, PieChart, Pie, Cell, ResponsiveContainer } from 'recharts';
import { TrendingUp, TrendingDown, DollarSign, Users, Target, Clock, Phone, Mail, ShoppingCart, Eye, MessageCircle, Heart, Share2, UserPlus, RefreshCw, Wifi, WifiOff } from 'lucide-react';

const Dashboard = () => {
  const [selectedDepartment, setSelectedDepartment] = useState('social_media');
  const [timeRange, setTimeRange] = useState('month');
  const [isLive, setIsLive] = useState(false);
  const [lastUpdated, setLastUpdated] = useState(new Date());
  const [refreshInterval, setRefreshInterval] = useState(30); // seconds
  const [isLoading, setIsLoading] = useState(false);

  // Real-time data fetching simulation
  const fetchRealTimeData = async () => {
    setIsLoading(true);
    
    // Simulate API calls to social media platforms
    const apiCalls = [
      simulateInstagramAPI(),
      simulateFacebookAPI(),
      simulateTwitterAPI(),
      simulateLinkedInAPI()
    ];
    
    try {
      await Promise.all(apiCalls);
      setLastUpdated(new Date());
    } catch (error) {
      console.error('Error fetching real-time data:', error);
    } finally {
      setIsLoading(false);
    }
  };

  // Simulate API calls (replace with actual API endpoints)
  const simulateInstagramAPI = async () => {
    // Example: await fetch('https://graph.instagram.com/me/media?fields=id,caption,timestamp,like_count,comments_count')
    return new Promise(resolve => setTimeout(resolve, 1000));
  };

  const simulateFacebookAPI = async () => {
    // Example: await fetch('https://graph.facebook.com/me/posts?fields=message,created_time,likes.summary(true),comments.summary(true)')
    return new Promise(resolve => setTimeout(resolve, 1200));
  };

  const simulateTwitterAPI = async () => {
    // Example: await fetch('https://api.twitter.com/2/users/me/tweets?tweet.fields=public_metrics')
    return new Promise(resolve => setTimeout(resolve, 800));
  };

  const simulateLinkedInAPI = async () => {
    // Example: await fetch('https://api.linkedin.com/v2/shares?q=owners&owners=urn:li:person:cindydawnpetty_ID')
    return new Promise(resolve => setTimeout(resolve, 1500));
  };

  // Auto-refresh functionality
  useEffect(() => {
    let interval;
    if (isLive) {
      interval = setInterval(() => {
        fetchRealTimeData();
      }, refreshInterval * 1000);
    }
    return () => clearInterval(interval);
  }, [isLive, refreshInterval]);

  // Manual refresh
  const handleManualRefresh = () => {
    fetchRealTimeData();
  };

  // Sample data for different departments
  const salesData = [
    { name: 'Jan', revenue: 45000, leads: 120, conversion: 15 },
    { name: 'Feb', revenue: 52000, leads: 140, conversion: 18 },
    { name: 'Mar', revenue: 48000, leads: 130, conversion: 16 },
    { name: 'Apr', revenue: 61000, leads: 160, conversion: 22 },
    { name: 'May', revenue: 55000, leads: 150, conversion: 20 },
    { name: 'Jun', revenue: 67000, leads: 180, conversion: 25 }
  ];

  const marketingData = [
    { name: 'Website', visitors: 12500, conversions: 245 },
    { name: 'Social Media', visitors: 8900, conversions: 178 },
    { name: 'Email', visitors: 5600, conversions: 112 },
    { name: 'Paid Ads', visitors: 15200, conversions: 456 },
    { name: 'Organic', visitors: 9800, conversions: 196 }
  ];

  const socialMediaData = [
    { name: 'Mon', posts: 5, engagement: 1250, reach: 15000, messages: 45 },
    { name: 'Tue', posts: 3, engagement: 980, reach: 12000, messages: 38 },
    { name: 'Wed', posts: 4, engagement: 1420, reach: 18000, messages: 52 },
    { name: 'Thu', posts: 6, engagement: 1680, reach: 20000, messages: 61 },
    { name: 'Fri', posts: 4, engagement: 1340, reach: 16500, messages: 43 },
    { name: 'Sat', posts: 2, engagement: 890, reach: 11000, messages: 28 },
    { name: 'Sun', posts: 3, engagement: 1150, reach: 14000, messages: 35 }
  ];

  const messagingData = [
    { platform: 'Instagram DM', messages: 245, response_time: 1.2, satisfaction: 4.5 },
    { platform: 'Facebook Messenger', messages: 189, response_time: 2.1, satisfaction: 4.2 },
    { platform: 'Twitter DM', messages: 98, response_time: 0.8, satisfaction: 4.6 },
    { platform: 'WhatsApp Business', messages: 156, response_time: 1.5, satisfaction: 4.4 },
    { platform: 'LinkedIn Messages', messages: 67, response_time: 3.2, satisfaction: 4.1 }
  ];

  const departmentKPIs = {
    social_media: {
      title: 'Social Media Performance',
      metrics: [
        { label: 'Total Reach', value: '156K', change: '+18%', trend: 'up', icon: Eye },
        { label: 'Engagement Rate', value: '4.2%', change: '+0.8%', trend: 'up', icon: Heart },
        { label: 'New Followers', value: '1,247', change: '+25%', trend: 'up', icon: UserPlus },
        { label: 'Messages Received', value: '755', change: '+12%', trend: 'up', icon: MessageCircle }
      ]
    },
    messaging: {
      title: 'Messaging & Communication',
      metrics: [
        { label: 'Messages Handled', value: '755', change: '+8%', trend: 'up', icon: MessageCircle },
        { label: 'Avg Response Time', value: '1.7h', change: '-20%', trend: 'up', icon: Clock },
        { label: 'Customer Satisfaction', value: '4.36', change: '+5%', trend: 'up', icon: Users },
        { label: 'Resolution Rate', value: '92%', change: '+3%', trend: 'up', icon: Target }
      ]
    },
    sales: {
      title: 'Sales Performance',
      metrics: [
        { label: 'Monthly Revenue', value: '$67,000', change: '+12%', trend: 'up', icon: DollarSign },
        { label: 'New Leads', value: '180', change: '+20%', trend: 'up', icon: Users },
        { label: 'Conversion Rate', value: '25%', change: '+5%', trend: 'up', icon: Target },
        { label: 'Avg Deal Size', value: '$2,680', change: '-3%', trend: 'down', icon: DollarSign }
      ]
    },
    marketing: {
      title: 'Marketing Analytics',
      metrics: [
        { label: 'Website Traffic', value: '52,100', change: '+8%', trend: 'up', icon: Eye },
        { label: 'Lead Generation', value: '1,187', change: '+15%', trend: 'up', icon: Users },
        { label: 'Cost per Lead', value: '$45', change: '-12%', trend: 'up', icon: DollarSign },
        { label: 'Email Open Rate', value: '24.5%', change: '+2%', trend: 'up', icon: Mail }
      ]
    },
    customer_service: {
      title: 'Customer Service',
      metrics: [
        { label: 'Tickets Resolved', value: '228', change: '+5%', trend: 'up', icon: Target },
        { label: 'Avg Response Time', value: '2.3h', change: '-15%', trend: 'up', icon: Clock },
        { label: 'Customer Satisfaction', value: '4.22', change: '+3%', trend: 'up', icon: Users },
        { label: 'First Call Resolution', value: '78%', change: '+7%', trend: 'up', icon: Phone }
      ]
    },
    operations: {
      title: 'Operations',
      metrics: [
        { label: 'Production Output', value: '1,245', change: '+6%', trend: 'up', icon: Target },
        { label: 'Quality Score', value: '96.5%', change: '+1%', trend: 'up', icon: Target },
        { label: 'Downtime', value: '0.8%', change: '-25%', trend: 'up', icon: Clock },
        { label: 'Cost per Unit', value: '$12.40', change: '-8%', trend: 'up', icon: DollarSign }
      ]
    }
  };

  const KPICard = ({ metric }) => {
    const Icon = metric.icon;
    return (
      <div className="bg-white rounded-lg shadow-md p-6 border-l-4 border-blue-500">
        <div className="flex items-center justify-between">
          <div>
            <p className="text-sm font-medium text-gray-600">{metric.label}</p>
            <p className="text-2xl font-bold text-gray-900">{metric.value}</p>
          </div>
          <div className="flex items-center">
            <Icon className="w-8 h-8 text-blue-500 mr-2" />
            <span className={`flex items-center text-sm font-medium ${
              metric.trend === 'up' ? 'text-green-600' : 'text-red-600'
            }`}>
              {metric.trend === 'up' ? <TrendingUp className="w-4 h-4 mr-1" /> : <TrendingDown className="w-4 h-4 mr-1" />}
              {metric.change}
            </span>
          </div>
        </div>
      </div>
    );
  };

  const getChartData = () => {
    switch (selectedDepartment) {
      case 'social_media':
        return socialMediaData;
      case 'messaging':
        return messagingData;
      case 'sales':
        return salesData;
      case 'marketing':
        return marketingData;
      default:
        return socialMediaData;
    }
  };

  const pieData = [
    { name: 'Instagram', value: 35, color: '#E4405F' },
    { name: 'Facebook', value: 28, color: '#1877F2' },
    { name: 'Twitter', value: 15, color: '#1DA1F2' },
    { name: 'LinkedIn', value: 12, color: '#0A66C2' },
    { name: 'TikTok', value: 10, color: '#000000' }
  ];

  return (
    <div className="min-h-screen bg-gray-50 p-6">
      <div className="max-w-7xl mx-auto">
        {/* Header with Real-time Controls */}
        <div className="mb-8">
          <div className="flex justify-between items-start">
            <div>
              <h1 className="text-3xl font-bold text-gray-900 mb-2">Business KPI Dashboard</h1>
              <p className="text-gray-600">Track your key performance indicators across all departments</p>
            </div>
            <div className="flex items-center space-x-4">
              <div className="flex items-center space-x-2">
                <div className={`w-3 h-3 rounded-full ${isLive ? 'bg-green-500 animate-pulse' : 'bg-gray-400'}`}></div>
                <span className="text-sm text-gray-600">
                  {isLive ? 'Live' : 'Offline'} | Updated: {lastUpdated.toLocaleTimeString()}
                </span>
              </div>
              <button
                onClick={handleManualRefresh}
                disabled={isLoading}
                className={`flex items-center space-x-2 px-4 py-2 rounded-lg ${
                  isLoading ? 'bg-gray-300 cursor-not-allowed' : 'bg-blue-500 hover:bg-blue-600'
                } text-white transition-colors`}
              >
                <RefreshCw className={`w-4 h-4 ${isLoading ? 'animate-spin' : ''}`} />
                <span>Refresh</span>
              </button>
            </div>
          </div>
        </div>

        {/* Real-time Controls */}
        <div className="bg-white rounded-lg shadow-md p-6 mb-8">
          <h3 className="text-lg font-semibold text-gray-900 mb-4">Real-time Settings</h3>
          <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
            <div>
              <label className="flex items-center space-x-3">
                <div className="relative">
                  <input
                    type="checkbox"
                    checked={isLive}
                    onChange={(e) => setIsLive(e.target.checked)}
                    className="sr-only"
                  />
                  <div className={`w-10 h-6 rounded-full ${isLive ? 'bg-green-500' : 'bg-gray-300'} relative transition-colors`}>
                    <div className={`w-4 h-4 bg-white rounded-full absolute top-1 transition-transform ${isLive ? 'translate-x-5' : 'translate-x-1'}`}></div>
                  </div>
                </div>
                <span className="text-sm font-medium text-gray-700">Enable Live Updates</span>
              </label>
            </div>
            <div>
              <label className="block text-sm font-medium text-gray-700 mb-2">Refresh Interval</label>
              <select
                value={refreshInterval}
                onChange={(e) => setRefreshInterval(parseInt(e.target.value))}
                className="border border-gray-300 rounded-md px-3 py-2 bg-white text-sm"
                disabled={!isLive}
              >
                <option value={15}>15 seconds</option>
                <option value={30}>30 seconds</option>
                <option value={60}>1 minute</option>
                <option value={300}>5 minutes</option>
                <option value={900}>15 minutes</option>
              </select>
            </div>
            <div>
              <label className="block text-sm font-medium text-gray-700 mb-2">API Status</label>
              <div className="flex items-center space-x-2">
                {isLive ? <Wifi className="w-4 h-4 text-green-500" /> : <WifiOff className="w-4 h-4 text-gray-400" />}
                <span className="text-sm text-gray-600">
                  {isLive ? 'Connected' : 'Disconnected'}
                </span>
              </div>
            </div>
          </div>
        </div>

        {/* Controls */}
        <div className="flex flex-wrap gap-4 mb-8">
          <div>
            <label className="block text-sm font-medium text-gray-700 mb-2">Department</label>
            <select 
              value={selectedDepartment} 
              onChange={(e) => setSelectedDepartment(e.target.value)}
              className="border border-gray-300 rounded-md px-3 py-2 bg-white"
            >
              <option value="social_media">Social Media</option>
              <option value="messaging">Messaging</option>
              <option value="sales">Sales</option>
              <option value="marketing">Marketing</option>
              <option value="customer_service">Customer Service</option>
              <option value="operations">Operations</option>
            </select>
          </div>
          <div>
            <label className="block text-sm font-medium text-gray-700 mb-2">Time Range</label>
            <select 
              value={timeRange} 
              onChange={(e) => setTimeRange(e.target.value)}
              className="border border-gray-300 rounded-md px-3 py-2 bg-white"
            >
              <option value="week">Last Week</option>
              <option value="month">Last Month</option>
              <option value="quarter">Last Quarter</option>
              <option value="year">Last Year</option>
            </select>
          </div>
        </div>

        {/* KPI Cards */}
        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6 mb-8">
          {departmentKPIs[selectedDepartment].metrics.map((metric, index) => (
            <KPICard key={index} metric={metric} />
          ))}
        </div>

        {/* Charts Section */}
        <div className="grid grid-cols-1 lg:grid-cols-2 gap-6 mb-8">
          {/* Main Chart */}
          <div className="bg-white rounded-lg shadow-md p-6">
            <h3 className="text-lg font-semibold text-gray-900 mb-4">
              {departmentKPIs[selectedDepartment].title} Trend
            </h3>
            <ResponsiveContainer width="100%" height={300}>
              {selectedDepartment === 'social_media' ? (
                <LineChart data={getChartData()}>
                  <CartesianGrid strokeDasharray="3 3" />
                  <XAxis dataKey="name" />
                  <YAxis />
                  <Tooltip />
                  <Line type="monotone" dataKey="engagement" stroke="#E4405F" strokeWidth={2} />
                  <Line type="monotone" dataKey="reach" stroke="#1877F2" strokeWidth={2} />
                </LineChart>
              ) : selectedDepartment === 'messaging' ? (
                <BarChart data={getChartData()}>
                  <CartesianGrid strokeDasharray="3 3" />
                  <XAxis dataKey="platform" />
                  <YAxis />
                  <Tooltip />
                  <Bar dataKey="messages" fill="#3B82F6" />
                </BarChart>
              ) : selectedDepartment === 'sales' ? (
                <LineChart data={getChartData()}>
                  <CartesianGrid strokeDasharray="3 3" />
                  <XAxis dataKey="name" />
                  <YAxis />
                  <Tooltip />
                  <Line type="monotone" dataKey="revenue" stroke="#3B82F6" strokeWidth={2} />
                </LineChart>
              ) : selectedDepartment === 'marketing' ? (
                <BarChart data={getChartData()}>
                  <CartesianGrid strokeDasharray="3 3" />
                  <XAxis dataKey="name" />
                  <YAxis />
                  <Tooltip />
                  <Bar dataKey="visitors" fill="#3B82F6" />
                </BarChart>
              ) : (
                <BarChart data={getChartData()}>
                  <CartesianGrid strokeDasharray="3 3" />
                  <XAxis dataKey="name" />
                  <YAxis />
                  <Tooltip />
                  <Bar dataKey="tickets" fill="#3B82F6" />
                  <Bar dataKey="resolved" fill="#10B981" />
                </BarChart>
              )}
            </ResponsiveContainer>
          </div>

          {/* Pie Chart */}
          <div className="bg-white rounded-lg shadow-md p-6">
            <h3 className="text-lg font-semibold text-gray-900 mb-4">Platform Distribution</h3>
            <ResponsiveContainer width="100%" height={300}>
              <PieChart>
                <Pie
                  data={pieData}
                  cx="50%"
                  cy="50%"
                  outerRadius={80}
                  dataKey="value"
                  label={({ name, percent }) => `${name} ${(percent * 100).toFixed(0)}%`}
                >
                  {pieData.map((entry, index) => (
                    <Cell key={`cell-${index}`} fill={entry.color} />
                  ))}
                </Pie>
                <Tooltip />
              </PieChart>
            </ResponsiveContainer>
          </div>
        </div>

        {/* Additional Metrics Table */}
        <div className="bg-white rounded-lg shadow-md p-6 mb-8">
          <div className="flex justify-between items-center mb-4">
            <h3 className="text-lg font-semibold text-gray-900">Detailed Metrics</h3>
            {isLoading && (
              <div className="flex items-center space-x-2 text-blue-500">
                <RefreshCw className="w-4 h-4 animate-spin" />
                <span className="text-sm">Updating...</span>
              </div>
            )}
          </div>
          <div className="overflow-x-auto">
            <table className="min-w-full divide-y divide-gray-200">
              <thead className="bg-gray-50">
                <tr>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Metric</th>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Current</th>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Previous</th>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Change</th>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Status</th>
                </tr>
              </thead>
              <tbody className="bg-white divide-y divide-gray-200">
                {departmentKPIs[selectedDepartment].metrics.map((metric, index) => (
                  <tr key={index} className={isLoading ? 'opacity-50' : ''}>
                    <td className="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">{metric.label}</td>
                    <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-900">{metric.value}</td>
                    <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-500">Previous Value</td>
                    <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-900">{metric.change}</td>
                    <td className="px-6 py-4 whitespace-nowrap">
                      <span className={`inline-flex px-2 py-1 text-xs font-semibold rounded-full ${
                        metric.trend === 'up' ? 'bg-green-100 text-green-800' : 'bg-red-100 text-red-800'
                      }`}>
                        {metric.trend === 'up' ? 'Good' : 'Needs Attention'}
                      </span>
                    </td>
                  </tr>
                ))}
              </tbody>
            </table>
          </div>
        </div>

        {/* API Integration Guide */}
        <div className="bg-white rounded-lg shadow-md p-6">
          <h3 className="text-lg font-semibold text-gray-900 mb-4">API Integration Setup</h3>
          <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
            <div>
              <h4 className="font-medium text-gray-900 mb-2">Social Media APIs</h4>
              <ul className="space-y-2 text-sm text-gray-600">
                <li>• <strong>Instagram:</strong> Graph API for posts, stories, insights</li>
                <li>• <strong>Facebook:</strong> Graph API for page metrics, posts</li>
                <li>• <strong>Twitter:</strong> API v2 for tweets, engagement</li>
                <li>• <strong>LinkedIn:</strong> Marketing API for company pages</li>
              </ul>
            </div>
            <div>
              <h4 className="font-medium text-gray-900 mb-2">Implementation Steps</h4>
              <ol className="space-y-2 text-sm text-gray-600">
                <li>1. Get API keys from each platform</li>
                <li>2. Set up authentication (OAuth 2.0)</li>
                <li>3. Replace simulation functions with real API calls</li>
                <li>4. Configure webhooks for instant updates</li>
                <li>5. Set up error handling and rate limiting</li>
              </ol>
            </div>
          </div>
        </div>
      </div>
    </div>
  );
};

export default Dashboard;await fetch('https://api.twitter.com/2/users/me/tweets?tweet.fields=public_metrics')https://www.facebook.com/cindy.d.mosley.7?mibextid=ZbWKwL/public/artifacts/ef1be513-8769-44c1-8e88-6d3c4bad91c3import React, { useState } from 'react';
import { BarChart, Bar, XAxis, YAxis, CartesianGrid, Tooltip, LineChart, Line, PieChart, Pie, Cell, ResponsiveContainer } from 'recharts';
import { TrendingUp, TrendingDown, DollarSign, Users, Target, Clock, Phone, Mail, ShoppingCart, Eye, MessageCircle, Heart, Share2, UserPlus } from 'lucide-react';

const Dashboard = () => {
  const [selectedDepartment, setSelectedDepartment] = useState('social_media');
  const [timeRange, setTimeRange] = useState('month');

  // Sample data for different departments
  const salesData = [
    { name: 'Jan', revenue: 45000, leads: 120, conversion: 15 },
    { name: 'Feb', revenue: 52000, leads: 140, conversion: 18 },
    { name: 'Mar', revenue: 48000, leads: 130, conversion: 16 },
    { name: 'Apr', revenue: 61000, leads: 160, conversion: 22 },
    { name: 'May', revenue: 55000, leads: 150, conversion: 20 },
    { name: 'Jun', revenue: 67000, leads: 180, conversion: 25 }
  ];

  const marketingData = [
    { name: 'Website', visitors: 12500, conversions: 245 },
    { name: 'Social Media', visitors: 8900, conversions: 178 },
    { name: 'Email', visitors: 5600, conversions: 112 },
    { name: 'Paid Ads', visitors: 15200, conversions: 456 },
    { name: 'Organic', visitors: 9800, conversions: 196 }
  ];

  const socialMediaData = [
    { name: 'Mon', posts: 5, engagement: 1250, reach: 15000, messages: 45 },
    { name: 'Tue', posts: 3, engagement: 980, reach: 12000, messages: 38 },
    { name: 'Wed', posts: 4, engagement: 1420, reach: 18000, messages: 52 },
    { name: 'Thu', posts: 6, engagement: 1680, reach: 20000, messages: 61 },
    { name: 'Fri', posts: 4, engagement: 1340, reach: 16500, messages: 43 },
    { name: 'Sat', posts: 2, engagement: 890, reach: 11000, messages: 28 },
    { name: 'Sun', posts: 3, engagement: 1150, reach: 14000, messages: 35 }
  ];

  const messagingData = [
    { platform: 'Instagram DM', messages: 245, response_time: 1.2, satisfaction: 4.5 },
    { platform: 'Facebook Messenger', messages: 189, response_time: 2.1, satisfaction: 4.2 },
    { platform: 'Twitter DM', messages: 98, response_time: 0.8, satisfaction: 4.6 },
    { platform: 'WhatsApp Business', messages: 156, response_time: 1.5, satisfaction: 4.4 },
    { platform: 'LinkedIn Messages', messages: 67, response_time: 3.2, satisfaction: 4.1 }
  ];

  const departmentKPIs = {
    social_media: {
      title: 'Social Media Performance',
      metrics: [
        { label: 'Total Reach', value: '156K', change: '+18%', trend: 'up', icon: Eye },
        { label: 'Engagement Rate', value: '4.2%', change: '+0.8%', trend: 'up', icon: Heart },
        { label: 'New Followers', value: '1,247', change: '+25%', trend: 'up', icon: UserPlus },
        { label: 'Messages Received', value: '755', change: '+12%', trend: 'up', icon: MessageCircle }
      ]
    },
    messaging: {
      title: 'Messaging & Communication',
      metrics: [
        { label: 'Messages Handled', value: '755', change: '+8%', trend: 'up', icon: MessageCircle },
        { label: 'Avg Response Time', value: '1.7h', change: '-20%', trend: 'up', icon: Clock },
        { label: 'Customer Satisfaction', value: '4.36', change: '+5%', trend: 'up', icon: Users },
        { label: 'Resolution Rate', value: '92%', change: '+3%', trend: 'up', icon: Target }
      ]
    },
    sales: {
      title: 'Sales Performance',
      metrics: [
        { label: 'Monthly Revenue', value: '$67,000', change: '+12%', trend: 'up', icon: DollarSign },
        { label: 'New Leads', value: '180', change: '+20%', trend: 'up', icon: Users },
        { label: 'Conversion Rate', value: '25%', change: '+5%', trend: 'up', icon: Target },
        { label: 'Avg Deal Size', value: '$2,680', change: '-3%', trend: 'down', icon: DollarSign }
      ]
    },
    marketing: {
      title: 'Marketing Analytics',
      metrics: [
        { label: 'Website Traffic', value: '52,100', change: '+8%', trend: 'up', icon: Eye },
        { label: 'Lead Generation', value: '1,187', change: '+15%', trend: 'up', icon: Users },
        { label: 'Cost per Lead', value: '$45', change: '-12%', trend: 'up', icon: DollarSign },
        { label: 'Email Open Rate', value: '24.5%', change: '+2%', trend: 'up', icon: Mail }
      ]
    },
    customer_service: {
      title: 'Customer Service',
      metrics: [
        { label: 'Tickets Resolved', value: '228', change: '+5%', trend: 'up', icon: Target },
        { label: 'Avg Response Time', value: '2.3h', change: '-15%', trend: 'up', icon: Clock },
        { label: 'Customer Satisfaction', value: '4.22', change: '+3%', trend: 'up', icon: Users },
        { label: 'First Call Resolution', value: '78%', change: '+7%', trend: 'up', icon: Phone }
      ]
    },
    operations: {
      title: 'Operations',
      metrics: [
        { label: 'Production Output', value: '1,245', change: '+6%', trend: 'up', icon: Target },
        { label: 'Quality Score', value: '96.5%', change: '+1%', trend: 'up', icon: Target },
        { label: 'Downtime', value: '0.8%', change: '-25%', trend: 'up', icon: Clock },
        { label: 'Cost per Unit', value: '$12.40', change: '-8%', trend: 'up', icon: DollarSign }
      ]
    }
  };

  const KPICard = ({ metric }) => {
    const Icon = metric.icon;
    return (
      <div className="bg-white rounded-lg shadow-md p-6 border-l-4 border-blue-500">
        <div className="flex items-center justify-between">
          <div>
            <p className="text-sm font-medium text-gray-600">{metric.label}</p>
            <p className="text-2xl font-bold text-gray-900">{metric.value}</p>
          </div>
          <div className="flex items-center">
            <Icon className="w-8 h-8 text-blue-500 mr-2" />
            <span className={`flex items-center text-sm font-medium ${
              metric.trend === 'up' ? 'text-green-600' : 'text-red-600'
            }`}>
              {metric.trend === 'up' ? <TrendingUp className="w-4 h-4 mr-1" /> : <TrendingDown className="w-4 h-4 mr-1" />}
              {metric.change}
            </span>
          </div>
        </div>
      </div>
    );
  };

  const getChartData = () => {
    switch (selectedDepartment) {
      case 'social_media':
        return socialMediaData;
      case 'messaging':
        return messagingData;
      case 'sales':
        return salesData;
      case 'marketing':
        return marketingData;
      default:
        return socialMediaData;
    }
  };

  const pieData = [
    { name: 'Instagram', value: 35, color: '#E4405F' },
    { name: 'Facebook', value: 28, color: '#1877F2' },
    { name: 'Twitter', value: 15, color: '#1DA1F2' },
    { name: 'LinkedIn', value: 12, color: '#0A66C2' },
    { name: 'TikTok', value: 10, color: '#000000' }
  ];

  return (
    <div className="min-h-screen bg-gray-50 p-6">
      <div className="max-w-7xl mx-auto">
        {/* Header */}
        <div className="mb-8">
          <h1 className="text-3xl font-bold text-gray-900 mb-2">Business KPI Dashboard</h1>
          <p className="text-gray-600">Track your key performance indicators across all departments</p>
        </div>

        {/* Controls */}
        <div className="flex flex-wrap gap-4 mb-8">
          <div>
            <label className="block text-sm font-medium text-gray-700 mb-2">Department</label>
            <select 
              value={selectedDepartment} 
              onChange={(e) => setSelectedDepartment(e.target.value)}
              className="border border-gray-300 rounded-md px-3 py-2 bg-white"
            >
              <option value="social_media">Social Media</option>
              <option value="messaging">Messaging</option>
              <option value="sales">Sales</option>
              <option value="marketing">Marketing</option>
              <option value="customer_service">Customer Service</option>
              <option value="operations">Operations</option>
            </select>
          </div>
          <div>
            <label className="block text-sm font-medium text-gray-700 mb-2">Time Range</label>
            <select 
              value={timeRange} 
              onChange={(e) => setTimeRange(e.target.value)}
              className="border border-gray-300 rounded-md px-3 py-2 bg-white"
            >
              <option value="week">Last Week</option>
              <option value="month">Last Month</option>
              <option value="quarter">Last Quarter</option>
              <option value="year">Last Year</option>
            </select>
          </div>
        </div>

        {/* KPI Cards */}
        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6 mb-8">
          {departmentKPIs[selectedDepartment].metrics.map((metric, index) => (
            <KPICard key={index} metric={metric} />
          ))}
        </div>

        {/* Charts Section */}
        <div className="grid grid-cols-1 lg:grid-cols-2 gap-6 mb-8">
          {/* Main Chart */}
          <div className="bg-white rounded-lg shadow-md p-6">
            <h3 className="text-lg font-semibold text-gray-900 mb-4">
              {departmentKPIs[selectedDepartment].title} Trend
            </h3>
            <ResponsiveContainer width="100%" height={300}>
              {selectedDepartment === 'social_media' ? (
                <LineChart data={getChartData()}>
                  <CartesianGrid strokeDasharray="3 3" />
                  <XAxis dataKey="name" />
                  <YAxis />
                  <Tooltip />
                  <Line type="monotone" dataKey="engagement" stroke="#E4405F" strokeWidth={2} />
                  <Line type="monotone" dataKey="reach" stroke="#1877F2" strokeWidth={2} />
                </LineChart>
              ) : selectedDepartment === 'messaging' ? (
                <BarChart data={getChartData()}>
                  <CartesianGrid strokeDasharray="3 3" />
                  <XAxis dataKey="platform" />
                  <YAxis />
                  <Tooltip />
                  <Bar dataKey="messages" fill="#3B82F6" />
                </BarChart>
              ) : selectedDepartment === 'sales' ? (
                <LineChart data={getChartData()}>
                  <CartesianGrid strokeDasharray="3 3" />
                  <XAxis dataKey="name" />
                  <YAxis />
                  <Tooltip />
                  <Line type="monotone" dataKey="revenue" stroke="#3B82F6" strokeWidth={2} />
                </LineChart>
              ) : selectedDepartment === 'marketing' ? (
                <BarChart data={getChartData()}>
                  <CartesianGrid strokeDasharray="3 3" />
                  <XAxis dataKey="name" />
                  <YAxis />
                  <Tooltip />
                  <Bar dataKey="visitors" fill="#3B82F6" />
                </BarChart>
              ) : (
                <BarChart data={getChartData()}>
                  <CartesianGrid strokeDasharray="3 3" />
                  <XAxis dataKey="name" />
                  <YAxis />
                  <Tooltip />
                  <Bar dataKey="tickets" fill="#3B82F6" />
                  <Bar dataKey="resolved" fill="#10B981" />
                </BarChart>
              )}
            </ResponsiveContainer>
          </div>

          {/* Pie Chart */}
          <div className="bg-white rounded-lg shadow-md p-6">
            <h3 className="text-lg font-semibold text-gray-900 mb-4">Platform Distribution</h3>
            <ResponsiveContainer width="100%" height={300}>
              <PieChart>
                <Pie
                  data={pieData}
                  cx="50%"
                  cy="50%"
                  outerRadius={80}
                  dataKey="value"
                  label={({ name, percent }) => `${name} ${(percent * 100).toFixed(0)}%`}
                >
                  {pieData.map((entry, index) => (
                    <Cell key={`cell-${index}`} fill={entry.color} />
                  ))}
                </Pie>
                <Tooltip />
              </PieChart>
            </ResponsiveContainer>
          </div>
        </div>

        {/* Additional Metrics Table */}
        <div className="bg-white rounded-lg shadow-md p-6">
          <h3 className="text-lg font-semibold text-gray-900 mb-4">Detailed Metrics</h3>
          <div className="overflow-x-auto">
            <table className="min-w-full divide-y divide-gray-200">
              <thead className="bg-gray-50">
                <tr>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Metric</th>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Current</th>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Previous</th>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Change</th>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Status</th>
                </tr>
              </thead>
              <tbody className="bg-white divide-y divide-gray-200">
                {departmentKPIs[selectedDepartment].metrics.map((metric, index) => (
                  <tr key={index}>
                    <td className="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">{metric.label}</td>
                    <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-900">{metric.value}</td>
                    <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-500">Previous Value</td>
                    <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-900">{metric.change}</td>
                    <td className="px-6 py-4 whitespace-nowrap">
                      <span className={`inline-flex px-2 py-1 text-xs font-semibold rounded-full ${
                        metric.trend === 'up' ? 'bg-green-100 text-green-800' : 'bg-red-100 text-red-800'
                      }`}>
                        {metric.trend === 'up' ? 'Good' : 'Needs Attention'}
                      </span>
                    </td>
                  </tr>
                ))}
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>
  );
};

export default Dashboard;
<h1 align="center">
  <br>
  <a href="https://github.com/ultrasecurity/Storm-Breaker"><img src=".imgs/1demo.png" alt="StormBreaker"></a>

</h1>

<h4 align="center">A Tool With Attractive Capabilities. </h4>

<p align="center">

  <a href="http://python.org">
    <img src="https://img.shields.io/badge/python-v3-blue">
  </a>
  <a href="https://php.net">
    <img src="https://img.shields.io/badge/php-7.4.4-green"
         alt="php">
  </a>

  <a href="https://en.wikipedia.org/wiki/Linux">
    <img src="https://img.shields.io/badge/Platform-Linux-red">
  </a>

</p>

![demo](.imgs/screen1.jpeg)

### Features:

- Obtain Device Information Without Any Permission !
- Access Location [SMARTPHONES]
- Access Webcam
- Access Microphone

<br>9363552693

### Update Log:

- Second (latest) Update on November 4th , 2022 .
- The overall structure of the tool is programmed from the beginning and is available as a web panel (in previous versions, the tool was available in the command line).
- Previous version's bugs fixed !
- Auto-download Ngrok Added !
- The templates have been optimized !
- Logs can be downloaded (NEW) !
- Clear log Added !
- It can be uploaded on a personal host (you won't have the Ngork problems anymore)
- You can start and stop the listener anytime ! (At will)
- Beautified user interface (NEW) !

> We have deleted Ngrok in the new version of Storm breaker and entrusted the user with running and sharing the localhost . So please note that Storm breaker runs a localhost for you and you have to start the Ngrok on your intended port yourself .
> <br>

#### Attention! :

> This version can be run on both local host and your personal domain and host . However , you can use it for both situations. If your country has suspended the Ngrok service, or your country's banned Ngrok, or your victim can't open the Ngrok link (for the reasons such as : He sees such a link as suspicious, Or if this service is suspended in his country) We suggest using the tool on your personal host and domain .
> <br>

## Default username and password:

- `username` : `admin`
- `password` : `admin`
- You can edit the config.php file to change the username and password .
  <br>

### Dependencies

**`Storm Breaker`** requires following programs to run properly -

- `php`
- `python3`
- `git`
- `Ngrok`

<!-- ![demo](.imgs/Work3.gif) -->
<br>

### Platforms Tested

- Kali Linux 2022
- macOS Big Sur / M1
- Termux (android)
- Personal host (direct admin and cPanel)
  <br>

### Installation On Kali Linux

```bash
$ git clone https://github.com/ultrasecurity/Storm-Breaker
$ cd Storm-Breaker
$ sudo bash install.sh
$ sudo python3 -m pip install -r requirements.txt
$ sudo python3 st.py
```

<br>

**`how to run personal host 👇`**

> Zip the contents of the storm-web folder completely and upload it to the public_html path .

> Note that the tool should not be opened in a path like this > yourdomain.com/st-web
> Instead , it should be opened purely in the public_html path (i.e. : don't just zip the storm-web folder itself, but manually zip its contents (the index.php file and other belongings should be in the public_html path)

#### Attention!:

> Note that to use this tool on your Localhost , You also need SSL . Because many of the tool's capabilities require SSL .

#### Attention!:

> To run ngrok on termux you need to enable your personal hotspot and cellular network.

</p>
