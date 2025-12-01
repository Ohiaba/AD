import React, { useState, useEffect, createContext, useContext } from 'react';
import { LineChart, Line, BarChart, Bar, PieChart, Pie, Cell, XAxis, YAxis, CartesianGrid, Tooltip, Legend, ResponsiveContainer, Area, AreaChart } from 'recharts';
import { Activity, Users, DollarSign, TrendingUp, RefreshCw, Settings, Bell, Menu } from 'lucide-react';

// Context for global dashboard state
const DashboardContext = createContext();

const useDashboard = () => {
  const context = useContext(DashboardContext);
  if (!context) throw new Error('useDashboard must be used within DashboardProvider');
  return context;
};

// Dashboard Provider Component
const DashboardProvider = ({ children }) => {
  const [metrics, setMetrics] = useState({
    totalUsers: 12847,
    revenue: 48923,
    activeUsers: 3421,
    growthRate: 23.5
  });

  const [realtimeData, setRealtimeData] = useState([]);
  const [salesData, setSalesData] = useState([]);
  const [userActivity, setUserActivity] = useState([]);
  const [categoryData, setCategoryData] = useState([]);
  const [isUpdating, setIsUpdating] = useState(false);
  const [lastUpdate, setLastUpdate] = useState(new Date());

  // Initialize data
  useEffect(() => {
    generateInitialData();
  }, []);

  // Real-time updates every 3 seconds
  useEffect(() => {
    const interval = setInterval(() => {
      updateRealtimeData();
    }, 3000);

    return () => clearInterval(interval);
  }, [realtimeData]);

  const generateInitialData = () => {
    const hours = ['00:00', '03:00', '06:00', '09:00', '12:00', '15:00', '18:00', '21:00'];
    const realtime = hours.map(hour => ({
      time: hour,
      users: Math.floor(Math.random() * 500) + 100,
      sessions: Math.floor(Math.random() * 300) + 50
    }));

    const sales = [
      { month: 'Jan', sales: 4200, target: 4000 },
      { month: 'Feb', sales: 3800, target: 4200 },
      { month: 'Mar', sales: 5100, target: 4500 },
      { month: 'Apr', sales: 4600, target: 4800 },
      { month: 'May', sales: 6200, target: 5000 },
      { month: 'Jun', sales: 5800, target: 5500 }
    ];

    const activity = [
      { name: 'Mon', mobile: 240, desktop: 400, tablet: 120 },
      { name: 'Tue', mobile: 300, desktop: 380, tablet: 150 },
      { name: 'Wed', mobile: 280, desktop: 420, tablet: 130 },
      { name: 'Thu', mobile: 350, desktop: 450, tablet: 180 },
      { name: 'Fri', mobile: 400, desktop: 500, tablet: 200 },
      { name: 'Sat', mobile: 320, desktop: 350, tablet: 140 },
      { name: 'Sun', mobile: 280, desktop: 300, tablet: 110 }
    ];

    const categories = [
      { name: 'Electronics', value: 4500, color: '#667eea' },
      { name: 'Fashion', value: 3200, color: '#764ba2' },
      { name: 'Home & Garden', value: 2800, color: '#f093fb' },
      { name: 'Sports', value: 2100, color: '#4facfe' },
      { name: 'Books', value: 1400, color: '#43e97b' }
    ];

    setRealtimeData(realtime);
    setSalesData(sales);
    setUserActivity(activity);
    setCategoryData(categories);
  };

  const updateRealtimeData = () => {
    setIsUpdating(true);
    
    setRealtimeData(prev => {
      const newData = [...prev];
      newData.shift();
      const now = new Date();
      newData.push({
        time: now.toLocaleTimeString('en-US', { hour: '2-digit', minute: '2-digit' }),
        users: Math.floor(Math.random() * 500) + 100,
        sessions: Math.floor(Math.random() * 300) + 50
      });
      return newData;
    });

    setMetrics(prev => ({
      totalUsers: prev.totalUsers + Math.floor(Math.random() * 10) - 5,
      revenue: prev.revenue + Math.floor(Math.random() * 100) - 50,
      activeUsers: Math.floor(Math.random() * 500) + 3000,
      growthRate: (Math.random() * 10 + 20).toFixed(1)
    }));

    setLastUpdate(new Date());
    setTimeout(() => setIsUpdating(false), 500);
  };

  const value = {
    metrics,
    realtimeData,
    salesData,
    userActivity,
    categoryData,
    isUpdating,
    lastUpdate,
    refreshData: generateInitialData
  };

  return (
    <DashboardContext.Provider value={value}>
      {children}
    </DashboardContext.Provider>
  );
};

// Metric Card Component
const MetricCard = ({ title, value, icon: Icon, color, change }) => {
  const { isUpdating } = useDashboard();
  
  return (
    <div className={`metric-card ${isUpdating ? 'updating' : ''}`} style={{ borderTopColor: color }}>
      <div className="metric-header">
        <span className="metric-title">{title}</span>
        <Icon size={24} color={color} />
      </div>
      <div className="metric-value">{value.toLocaleString()}</div>
      <div className="metric-change" style={{ color: change >= 0 ? '#43e97b' : '#f093fb' }}>
        <TrendingUp size={16} />
        <span>{change >= 0 ? '+' : ''}{change}%</span>
      </div>
    </div>
  );
};

// Chart Components
const RealtimeChart = () => {
  const { realtimeData } = useDashboard();

  return (
    <div className="chart-container">
      <h3 className="chart-title">Real-Time User Activity</h3>
      <ResponsiveContainer width="100%" height={300}>
        <AreaChart data={realtimeData}>
          <defs>
            <linearGradient id="colorUsers" x1="0" y1="0" x2="0" y2="1">
              <stop offset="5%" stopColor="#667eea" stopOpacity={0.8}/>
              <stop offset="95%" stopColor="#667eea" stopOpacity={0.1}/>
            </linearGradient>
            <linearGradient id="colorSessions" x1="0" y1="0" x2="0" y2="1">
              <stop offset="5%" stopColor="#764ba2" stopOpacity={0.8}/>
              <stop offset="95%" stopColor="#764ba2" stopOpacity={0.1}/>
            </linearGradient>
          </defs>
          <CartesianGrid strokeDasharray="3 3" stroke="#e0e0e0" />
          <XAxis dataKey="time" stroke="#666" />
          <YAxis stroke="#666" />
          <Tooltip 
            contentStyle={{ 
              background: 'rgba(255,255,255,0.95)', 
              border: '1px solid #667eea',
              borderRadius: '8px',
              boxShadow: '0 4px 12px rgba(0,0,0,0.1)'
            }} 
          />
          <Area type="monotone" dataKey="users" stroke="#667eea" fillOpacity={1} fill="url(#colorUsers)" />
          <Area type="monotone" dataKey="sessions" stroke="#764ba2" fillOpacity={1} fill="url(#colorSessions)" />
        </AreaChart>
      </ResponsiveContainer>
    </div>
  );
};

const SalesChart = () => {
  const { salesData } = useDashboard();

  return (
    <div className="chart-container">
      <h3 className="chart-title">Monthly Sales Performance</h3>
      <ResponsiveContainer width="100%" height={300}>
        <BarChart data={salesData}>
          <CartesianGrid strokeDasharray="3 3" stroke="#e0e0e0" />
          <XAxis dataKey="month" stroke="#666" />
          <YAxis stroke="#666" />
          <Tooltip 
            contentStyle={{ 
              background: 'rgba(255,255,255,0.95)', 
              border: '1px solid #667eea',
              borderRadius: '8px'
            }} 
          />
          <Legend />
          <Bar dataKey="sales" fill="#667eea" radius={[8, 8, 0, 0]} />
          <Bar dataKey="target" fill="#f093fb" radius={[8, 8, 0, 0]} />
        </BarChart>
      </ResponsiveContainer>
    </div>
  );
};

const ActivityChart = () => {
  const { userActivity } = useDashboard();

  return (
    <div className="chart-container">
      <h3 className="chart-title">Device Activity by Day</h3>
      <ResponsiveContainer width="100%" height={300}>
        <LineChart data={userActivity}>
          <CartesianGrid strokeDasharray="3 3" stroke="#e0e0e0" />
          <XAxis dataKey="name" stroke="#666" />
          <YAxis stroke="#666" />
          <Tooltip 
            contentStyle={{ 
              background: 'rgba(255,255,255,0.95)', 
              border: '1px solid #667eea',
              borderRadius: '8px'
            }} 
          />
          <Legend />
          <Line type="monotone" dataKey="mobile" stroke="#667eea" strokeWidth={3} dot={{ r: 4 }} />
          <Line type="monotone" dataKey="desktop" stroke="#764ba2" strokeWidth={3} dot={{ r: 4 }} />
          <Line type="monotone" dataKey="tablet" stroke="#f093fb" strokeWidth={3} dot={{ r: 4 }} />
        </LineChart>
      </ResponsiveContainer>
    </div>
  );
};

const CategoryChart = () => {
  const { categoryData } = useDashboard();

  return (
    <div className="chart-container">
      <h3 className="chart-title">Sales by Category</h3>
      <ResponsiveContainer width="100%" height={300}>
        <PieChart>
          <Pie
            data={categoryData}
            cx="50%"
            cy="50%"
            labelLine={false}
            label={({ name, percent }) => `${name}: ${(percent * 100).toFixed(0)}%`}
            outerRadius={100}
            fill="#8884d8"
            dataKey="value"
          >
            {categoryData.map((entry, index) => (
              <Cell key={`cell-${index}`} fill={entry.color} />
            ))}
          </Pie>
          <Tooltip />
        </PieChart>
      </ResponsiveContainer>
    </div>
  );
};

// Main Dashboard Component
const Dashboard = () => {
  const { metrics, isUpdating, lastUpdate, refreshData } = useDashboard();
  const [sidebarOpen, setSidebarOpen] = useState(false);

  return (
    <div className="dashboard">
      <header className="dashboard-header">
        <div className="header-left">
          <button className="menu-btn" onClick={() => setSidebarOpen(!sidebarOpen)}>
            <Menu size={24} />
          </button>
          <h1 className="dashboard-title">Analytics Dashboard</h1>
        </div>
        <div className="header-right">
          <div className="last-update">
            Last update: {lastUpdate.toLocaleTimeString()}
          </div>
          <button className="icon-btn" onClick={refreshData}>
            <RefreshCw size={20} className={isUpdating ? 'spinning' : ''} />
          </button>
          <button className="icon-btn">
            <Bell size={20} />
          </button>
          <button className="icon-btn">
            <Settings size={20} />
          </button>
        </div>
      </header>

      <div className="dashboard-content">
        <div className="metrics-grid">
          <MetricCard 
            title="Total Users" 
            value={metrics.totalUsers} 
            icon={Users}
            color="#667eea"
            change={5.2}
          />
          <MetricCard 
            title="Revenue" 
            value={metrics.revenue} 
            icon={DollarSign}
            color="#43e97b"
            change={12.8}
          />
          <MetricCard 
            title="Active Users" 
            value={metrics.activeUsers} 
            icon={Activity}
            color="#764ba2"
            change={-2.3}
          />
          <MetricCard 
            title="Growth Rate" 
            value={metrics.growthRate} 
            icon={TrendingUp}
            color="#f093fb"
            change={parseFloat(metrics.growthRate)}
          />
        </div>

        <div className="charts-grid">
          <div className="chart-full">
            <RealtimeChart />
          </div>
          <div className="chart-half">
            <SalesChart />
          </div>
          <div className="chart-half">
            <ActivityChart />
          </div>
          <div className="chart-full">
            <CategoryChart />
          </div>
        </div>
      </div>
    </div>
  );
};

// Main App Component
export default function App() {
  return (
    <DashboardProvider>
      <Dashboard />
      <style>{`
        * {
          margin: 0;
          padding: 0;
          box-sizing: border-box;
        }

        body {
          font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
          background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
          min-height: 100vh;
        }

        .dashboard {
          min-height: 100vh;
          padding: 20px;
        }

        .dashboard-header {
          background: white;
          border-radius: 16px;
          padding: 20px 30px;
          margin-bottom: 20px;
          display: flex;
          justify-content: space-between;
          align-items: center;
          box-shadow: 0 10px 30px rgba(0,0,0,0.1);
        }

        .header-left {
          display: flex;
          align-items: center;
          gap: 15px;
        }

        .menu-btn {
          background: none;
          border: none;
          cursor: pointer;
          padding: 8px;
          border-radius: 8px;
          transition: background 0.3s;
        }

        .menu-btn:hover {
          background: #f0f0f0;
        }

        .dashboard-title {
          font-size: 28px;
          font-weight: 700;
          background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
          -webkit-background-clip: text;
          -webkit-text-fill-color: transparent;
          background-clip: text;
        }

        .header-right {
          display: flex;
          align-items: center;
          gap: 15px;
        }

        .last-update {
          color: #666;
          font-size: 14px;
          padding: 8px 16px;
          background: #f5f5f5;
          border-radius: 20px;
        }

        .icon-btn {
          background: none;
          border: none;
          cursor: pointer;
          padding: 10px;
          border-radius: 50%;
          transition: all 0.3s;
          display: flex;
          align-items: center;
          justify-content: center;
        }

        .icon-btn:hover {
          background: #f0f0f0;
          transform: scale(1.1);
        }

        .spinning {
          animation: spin 1s linear infinite;
        }

        @keyframes spin {
          from { transform: rotate(0deg); }
          to { transform: rotate(360deg); }
        }

        .dashboard-content {
          display: flex;
          flex-direction: column;
          gap: 20px;
        }

        .metrics-grid {
          display: grid;
          grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
          gap: 20px;
        }

        .metric-card {
          background: white;
          padding: 25px;
          border-radius: 16px;
          border-top: 4px solid;
          box-shadow: 0 10px 30px rgba(0,0,0,0.1);
          transition: all 0.3s;
        }

        .metric-card:hover {
          transform: translateY(-5px);
          box-shadow: 0 15px 40px rgba(0,0,0,0.15);
        }

        .metric-card.updating {
          animation: pulse 0.5s ease-in-out;
        }

        @keyframes pulse {
          0%, 100% { transform: scale(1); }
          50% { transform: scale(1.02); }
        }

        .metric-header {
          display: flex;
          justify-content: space-between;
          align-items: center;
          margin-bottom: 15px;
        }

        .metric-title {
          color: #666;
          font-size: 14px;
          font-weight: 600;
          text-transform: uppercase;
          letter-spacing: 0.5px;
        }

        .metric-value {
          font-size: 32px;
          font-weight: 700;
          color: #333;
          margin-bottom: 10px;
        }

        .metric-change {
          display: flex;
          align-items: center;
          gap: 5px;
          font-size: 14px;
          font-weight: 600;
        }

        .charts-grid {
          display: grid;
          grid-template-columns: repeat(2, 1fr);
          gap: 20px;
        }

        .chart-full {
          grid-column: span 2;
        }

        .chart-half {
          grid-column: span 1;
        }

        .chart-container {
          background: white;
          padding: 25px;
          border-radius: 16px;
          box-shadow: 0 10px 30px rgba(0,0,0,0.1);
        }

        .chart-title {
          font-size: 18px;
          font-weight: 700;
          color: #333;
          margin-bottom: 20px;
          padding-bottom: 15px;
          border-bottom: 2px solid #f0f0f0;
        }

        @media (max-width: 1024px) {
          .charts-grid {
            grid-template-columns: 1fr;
          }

          .chart-full,
          .chart-half {
            grid-column: span 1;
          }
        }

        @media (max-width: 768px) {
          .dashboard {
            padding: 10px;
          }

          .dashboard-header {
            flex-direction: column;
            gap: 15px;
            padding: 15px;
          }

          .header-right {
            width: 100%;
            justify-content: space-between;
          }

          .dashboard-title {
            font-size: 20px;
          }

          .metrics-grid {
            grid-template-columns: 1fr;
          }

          .last-update {
            font-size: 12px;
          }
        }
      `}</style>
    </DashboardProvider>
  );
}
