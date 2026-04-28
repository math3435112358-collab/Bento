import React, { useState, useMemo } from 'react';
import { 
  Calendar, 
  ShoppingCart, 
  ClipboardList, 
  User, 
  CheckCircle, 
  AlertTriangle, 
  QrCode, 
  LayoutDashboard,
  ChevronRight,
  TrendingUp,
  Search
} from 'lucide-react';

// --- モックデータ ---
const MENU_ITEMS = [
  { id: 1, name: '幕の内弁当', price: 550, stock: 15, allergies: ['卵', '小麦', 'サバ'], description: '定番のおかずが詰まった栄養満点弁当。', category: '和食' },
  { id: 2, name: '若鶏の唐揚げ弁当', price: 500, stock: 40, allergies: ['小麦', '鶏肉'], description: 'ジューシーな唐揚げが大人気。', category: '洋食' },
  { id: 3, name: '特製ハンバーグ弁当', price: 600, stock: 8, allergies: ['乳', '卵', '小麦', '牛肉'], description: 'デミグラスソースたっぷりの手作り。', category: '洋食' },
  { id: 4, name: 'サバの味噌煮弁当', price: 520, stock: 20, allergies: ['小麦', 'サバ', '大豆'], description: '脂ののったサバを丁寧に煮込みました。', category: '和食' },
];

const INITIAL_ORDERS = [
  { id: 'ORD001', studentName: '田中 太郎', className: '1-A', itemName: '幕の内弁当', status: '未受取' },
  { id: 'ORD002', studentName: '佐藤 花子', className: '1-A', itemName: '唐揚げ弁当', status: '済' },
  { id: 'ORD003', studentName: '鈴木 一郎', className: '2-B', itemName: '幕の内弁当', status: '未受取' },
  { id: 'ORD004', studentName: '高橋 健二', className: '3-C', itemName: 'ハンバーグ弁当', status: '未受取' },
];

const App = () => {
  const [view, setView] = useState('user'); // 'user' | 'admin' | 'success'
  const [cart, setCart] = useState([]);
  const [confirmedOrder, setConfirmedOrder] = useState(null);

  // --- ロジック ---
  const addToCart = (item) => {
    if (item.stock > 0 && !cart.find(i => i.id === item.id)) {
      setCart([...cart, item]);
    }
  };

  const removeFromCart = (id) => {
    setCart(cart.filter(item => item.id !== id));
  };

  const executeOrder = () => {
    const orderId = `QR-${Math.random().toString(36).substr(2, 9).toUpperCase()}`;
    setConfirmedOrder({ id: orderId, items: cart });
    setCart([]);
    setView('success');
  };

  const totalPrice = cart.reduce((sum, item) => sum + item.price, 0);

  // --- コンポーネント: ユーザー画面 ---
  const UserView = () => (
    <div className="pb-24">
      <div className="flex items-center justify-between mb-6">
        <h2 className="text-xl font-bold text-slate-800 flex items-center gap-2">
          <Calendar className="text-blue-600" /> 4月28日(火) のメニュー
        </h2>
      </div>

      <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
        {MENU_ITEMS.map(item => (
          <div key={item.id} className="bg-white border border-slate-200 rounded-2xl p-4 shadow-sm hover:shadow-md transition-shadow">
            <div className="flex justify-between items-start mb-2">
              <span className="px-2 py-1 bg-blue-50 text-blue-600 text-xs font-bold rounded">{item.category}</span>
              <span className={`text-xs font-bold ${item.stock < 10 ? 'text-red-500' : 'text-slate-400'}`}>
                残り {item.stock}個
              </span>
            </div>
            <h3 className="text-lg font-bold text-slate-800 mb-1">{item.name}</h3>
            <p className="text-sm text-slate-500 mb-3 leading-relaxed">{item.description}</p>
            
            <div className="flex flex-wrap gap-1 mb-4">
              {item.allergies.map(a => (
                <span key={a} className="flex items-center gap-0.5 px-2 py-0.5 bg-amber-50 text-amber-700 text-[10px] rounded border border-amber-100">
                  <AlertTriangle size={10} /> {a}
                </span>
              ))}
            </div>

            <div className="flex items-center justify-between mt-auto">
              <span className="text-xl font-bold text-slate-900">¥{item.price.toLocaleString()}</span>
              <button 
                onClick={() => addToCart(item)}
                disabled={item.stock === 0 || cart.find(i => i.id === item.id)}
                className={`px-4 py-2 rounded-xl font-bold transition-colors ${
                  cart.find(i => i.id === item.id) 
                  ? 'bg-slate-100 text-slate-400 cursor-not-allowed'
                  : 'bg-blue-600 text-white hover:bg-blue-700'
                }`}
              >
                {cart.find(i => i.id === item.id) ? '追加済み' : 'カートに入れる'}
              </button>
            </div>
          </div>
        ))}
      </div>

      {/* 簡易カートフローティングバー */}
      {cart.length > 0 && (
        <div className="fixed bottom-6 left-4 right-4 bg-white border border-blue-100 shadow-2xl rounded-2xl p-4 flex items-center justify-between animate-in fade-in slide-in-from-bottom-4 duration-300">
          <div>
            <p className="text-xs text-slate-500 font-medium">現在の選択: {cart.length}点</p>
            <p className="text-lg font-bold text-slate-900">合計 ¥{totalPrice.toLocaleString()}</p>
          </div>
          <button 
            onClick={executeOrder}
            className="bg-blue-600 text-white px-8 py-3 rounded-xl font-bold shadow-lg shadow-blue-200 active:scale-95 transition-all"
          >
            注文を確定する
          </button>
        </div>
      )}
    </div>
  );

  // --- コンポーネント: 完了画面 ---
  const SuccessView = () => (
    <div className="flex flex-col items-center justify-center py-12 text-center">
      <div className="w-20 h-20 bg-green-100 text-green-600 rounded-full flex items-center justify-center mb-6">
        <CheckCircle size={48} />
      </div>
      <h2 className="text-2xl font-bold text-slate-800 mb-2">注文が完了しました！</h2>
      <p className="text-slate-500 mb-8">受け取り時に以下のQRコードを提示してください。</p>
      
      <div className="bg-white p-6 rounded-3xl shadow-xl border border-slate-100 mb-8">
        <div className="w-48 h-48 bg-slate-50 border-2 border-dashed border-slate-200 rounded-xl flex items-center justify-center">
          <QrCode size={120} className="text-slate-800" />
        </div>
        <p className="mt-4 font-mono font-bold text-lg text-slate-700">{confirmedOrder?.id}</p>
      </div>

      <button 
        onClick={() => setView('user')}
        className="text-blue-600 font-bold hover:underline"
      >
        メニューに戻る
      </button>
    </div>
  );

  // --- コンポーネント: 管理者画面 ---
  const AdminView = () => {
    const stats = [
      { label: '幕の内弁当', count: 25, trend: '+12%' },
      { label: '唐揚げ弁当', count: 42, trend: '+5%' },
      { label: 'ハンバーグ弁当', count: 18, trend: '-2%' },
    ];

    return (
      <div className="space-y-8">
        <section>
          <h3 className="text-sm font-bold text-slate-500 uppercase tracking-wider mb-4">本日の注文集計</h3>
          <div className="grid grid-cols-1 sm:grid-cols-3 gap-4">
            {stats.map(s => (
              <div key={s.label} className="bg-white p-4 rounded-2xl border border-slate-200 shadow-sm">
                <p className="text-slate-500 text-sm mb-1">{s.label}</p>
                <div className="flex items-end justify-between">
                  <span className="text-2xl font-bold text-slate-800">{s.count}食</span>
                  <span className={`text-xs font-bold ${s.trend.startsWith('+') ? 'text-green-500' : 'text-red-500'}`}>
                    {s.trend}
                  </span>
                </div>
              </div>
            ))}
          </div>
        </section>

        <section>
          <div className="flex items-center justify-between mb-4">
            <h3 className="text-sm font-bold text-slate-500 uppercase tracking-wider">クラス別配布リスト</h3>
            <div className="relative">
              <Search size={16} className="absolute left-3 top-1/2 -translate-y-1/2 text-slate-400" />
              <input type="text" placeholder="クラス・氏名検索" className="pl-9 pr-4 py-2 bg-slate-100 border-none rounded-lg text-sm w-48 focus:ring-2 focus:ring-blue-500" />
            </div>
          </div>
          <div className="bg-white border border-slate-200 rounded-2xl overflow-hidden shadow-sm">
            <table className="w-full text-left border-collapse">
              <thead>
                <tr className="bg-slate-50 text-slate-500 text-xs uppercase">
                  <th className="p-4 font-bold">クラス</th>
                  <th className="p-4 font-bold">氏名</th>
                  <th className="p-4 font-bold">注文内容</th>
                  <th className="p-4 font-bold">ステータス</th>
                </tr>
              </thead>
              <tbody className="text-sm divide-y divide-slate-100">
                {INITIAL_ORDERS.map(order => (
                  <tr key={order.id} className="hover:bg-slate-50 transition-colors">
                    <td className="p-4 font-bold text-slate-700">{order.className}</td>
                    <td className="p-4 text-slate-600">{order.studentName}</td>
                    <td className="p-4 text-slate-600">{order.itemName}</td>
                    <td className="p-4">
                      <span className={`px-2 py-1 rounded-full text-[10px] font-bold ${
                        order.status === '済' ? 'bg-green-50 text-green-600' : 'bg-amber-50 text-amber-600'
                      }`}>
                        {order.status}
                      </span>
                    </td>
                  </tr>
                ))}
              </tbody>
            </table>
          </div>
        </section>
      </div>
    );
  };

  return (
    <div className="min-h-screen bg-slate-50 font-sans text-slate-900">
      {/* ナビゲーションバー */}
      <header className="sticky top-0 z-10 bg-white/80 backdrop-blur-md border-b border-slate-200 px-4 py-4">
        <div className="max-w-3xl mx-auto flex items-center justify-between">
          <div className="flex items-center gap-2">
            <div className="bg-blue-600 p-2 rounded-lg">
              <ClipboardList className="text-white" size={20} />
            </div>
            <h1 className="font-black text-xl tracking-tight text-slate-800">学食予約 <span className="text-blue-600">SmartBento</span></h1>
          </div>
          
          <nav className="flex bg-slate-100 p-1 rounded-xl">
            <button 
              onClick={() => setView('user')}
              className={`flex items-center gap-1.5 px-4 py-1.5 rounded-lg text-sm font-bold transition-all ${view === 'user' || view === 'success' ? 'bg-white shadow-sm text-blue-600' : 'text-slate-500'}`}
            >
              <User size={16} /> 注文
            </button>
            <button 
              onClick={() => setView('admin')}
              className={`flex items-center gap-1.5 px-4 py-1.5 rounded-lg text-sm font-bold transition-all ${view === 'admin' ? 'bg-white shadow-sm text-blue-600' : 'text-slate-500'}`}
            >
              <LayoutDashboard size={16} /> 管理
            </button>
          </nav>
        </div>
      </header>

      {/* メインコンテンツ */}
      <main className="max-w-3xl mx-auto p-4 md:p-6">
        {view === 'user' && <UserView />}
        {view === 'admin' && <AdminView />}
        {view === 'success' && <SuccessView />}
      </main>
    </div>
  );
};

export default App;
