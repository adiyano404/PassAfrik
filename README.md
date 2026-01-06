<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PassAfrik - The Universal Pass for African Events</title>
    <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        * { font-family: 'Inter', sans-serif; }
        .gradient-bg { background: linear-gradient(135deg, #FF6B35 0%, #F7931E 50%, #FFD700 100%); }
        .gradient-text { background: linear-gradient(135deg, #FF6B35, #F7931E); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
        .card-hover { transition: all 0.3s ease; }
        .card-hover:hover { transform: translateY(-5px); box-shadow: 0 20px 40px rgba(0,0,0,0.1); }
        .sidebar-item.active { background: linear-gradient(135deg, #FF6B35 0%, #F7931E 100%); color: white; }
        .stat-card { background: linear-gradient(135deg, rgba(255,107,53,0.1), rgba(247,147,30,0.05)); }
        .country-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(120px, 1fr)); gap: 1rem; }
        .mobile-menu { transform: translateX(-100%); transition: transform 0.3s ease; }
        .mobile-menu.open { transform: translateX(0); }
        .modal-overlay { opacity: 0; transition: opacity 0.3s ease; }
        .modal-overlay.active { opacity: 1; }
        .modal-content { transform: scale(0.9); transition: transform 0.3s ease; }
        .modal-overlay.active .modal-content { transform: scale(1); }
        .toast { transform: translateX(100%); transition: transform 0.3s ease; }
        .toast.show { transform: translateX(0); }
        .spinner { border: 3px solid rgba(255,107,53,0.2); border-top-color: #FF6B35; border-radius: 50%; width: 24px; height: 24px; animation: spin 1s linear infinite; }
        @keyframes spin { to { transform: rotate(360deg); } }
    </style>
</head>
<body class="bg-gray-50">
    <div id="toastContainer" class="fixed top-20 right-4 z-[100] space-y-2"></div>
    
    <div id="loadingOverlay" class="fixed inset-0 bg-black/50 z-[200] hidden items-center justify-center">
        <div class="bg-white rounded-2xl p-8 flex flex-col items-center">
            <div class="spinner mb-4"></div>
            <p class="font-medium">Processing...</p>
        </div>
    </div>

    <div id="mobileMenuOverlay" class="fixed inset-0 bg-black/50 z-[60] hidden" onclick="closeMobileMenu()"></div>
    
    <div id="mobileMenu" class="mobile-menu fixed left-0 top-0 h-full w-72 bg-white z-[70] shadow-2xl">
        <div class="p-6">
            <div class="flex items-center justify-between mb-8">
                <div class="flex items-center">
                    <div class="w-10 h-10 gradient-bg rounded-xl flex items-center justify-center">
                        <i class="fas fa-passport text-white text-lg"></i>
                    </div>
                    <span class="ml-2 text-xl font-bold gradient-text">PassAfrik</span>
                </div>
                <button onclick="closeMobileMenu()" class="text-gray-500 hover:text-gray-700">
                    <i class="fas fa-times text-xl"></i>
                </button>
            </div>
            <nav class="space-y-2">
                <button onclick="showView('discover'); closeMobileMenu();" class="w-full text-left px-4 py-3 rounded-xl hover:bg-orange-50 text-gray-700 font-medium flex items-center">
                    <i class="fas fa-compass w-6"></i><span>Discover</span>
                </button>
                <button onclick="showView('promoter'); closeMobileMenu();" class="w-full text-left px-4 py-3 rounded-xl hover:bg-orange-50 text-gray-700 font-medium flex items-center">
                    <i class="fas fa-bullhorn w-6"></i><span>For Promoters</span>
                </button>
                <button onclick="showView('pricing'); closeMobileMenu();" class="w-full text-left px-4 py-3 rounded-xl hover:bg-orange-50 text-gray-700 font-medium flex items-center">
                    <i class="fas fa-tags w-6"></i><span>Pricing</span>
                </button>
            </nav>
            <div class="border-t mt-6 pt-6">
                <button onclick="showView('dashboard'); closeMobileMenu();" class="w-full py-3 gradient-bg text-white rounded-xl font-semibold">Get Started</button>
            </div>
        </div>
    </div>

    <nav class="bg-white shadow-lg fixed w-full z-50">
        <div class="max-w-7xl mx-auto px-4">
            <div class="flex justify-between items-center h-16">
                <div class="flex items-center space-x-4 md:space-x-8">
                    <button onclick="openMobileMenu()" class="md:hidden text-gray-700 hover:text-orange-500">
                        <i class="fas fa-bars text-xl"></i>
                    </button>
                    <div class="flex items-center cursor-pointer" onclick="showView('discover')">
                        <div class="w-10 h-10 gradient-bg rounded-xl flex items-center justify-center">
                            <i class="fas fa-passport text-white text-lg"></i>
                        </div>
                        <div class="ml-2">
                            <span class="text-xl font-bold gradient-text">PassAfrik</span>
                            <p class="text-xs text-gray-500 hidden md:block">The Universal Pass for African Events</p>
                        </div>
                    </div>
                    <div class="hidden md:flex space-x-6">
                        <button onclick="showView('discover')" class="text-gray-600 hover:text-orange-500 font-medium">Discover</button>
                        <button onclick="showView('promoter')" class="text-gray-600 hover:text-orange-500 font-medium">For Promoters</button>
                        <button onclick="showView('pricing')" class="text-gray-600 hover:text-orange-500 font-medium">Pricing</button>
                    </div>
                </div>
                <div class="flex items-center space-x-4">
                    <div class="relative">
                        <select id="countrySelect" onchange="changeCountry(this.value)" class="appearance-none bg-gray-100 rounded-lg px-3 py-2 pr-8 text-sm font-medium focus:outline-none">
                        </select>
                        <i class="fas fa-chevron-down absolute right-2 top-1/2 -translate-y-1/2 text-gray-400 text-xs pointer-events-none"></i>
                    </div>
                    <div class="flex bg-gray-100 rounded-lg p-1">
                        <button onclick="setLanguage('en')" id="lang-en" class="px-3 py-1 rounded-md text-sm font-medium bg-orange-500 text-white">EN</button>
                        <button onclick="setLanguage('fr')" id="lang-fr" class="px-3 py-1 rounded-md text-sm font-medium text-gray-600">FR</button>
                    </div>
                    <button onclick="showView('dashboard')" class="gradient-bg text-white px-4 py-2 rounded-lg font-medium hover:opacity-90">Get Started</button>
                </div>
            </div>
        </div>
    </nav> PassAfrik
PassAfrik - cross border ticketing platform
    <!-- Discover View -->
    <div id="view-discover" class="view pt-16">
        <div class="gradient-bg py-20">
            <div class="max-w-7xl mx-auto px-4 text-center">
                <div class="inline-flex items-center bg-white/20 backdrop-blur rounded-full px-4 py-2 mb-6">
                    <i class="fas fa-passport mr-2 text-white"></i>
                    <span class="text-white font-medium">The Universal Pass for African Events</span>
                </div>
                <h1 class="text-4xl md:text-6xl font-bold text-white mb-6">Discover Amazing Events Across Africa</h1>
                <p class="text-xl text-white/90 mb-8 max-w-2xl mx-auto">Your gateway to concerts, festivals, conferences, and more in 12 countries</p>
                <div class="flex flex-col md:flex-row gap-4 justify-center items-center max-w-2xl mx-auto">
                    <div class="relative flex-1 w-full">
                        <i class="fas fa-search absolute left-4 top-1/2 -translate-y-1/2 text-gray-400"></i>
                        <input type="text" id="searchInput" placeholder="Search events, artists, venues..." class="w-full pl-12 pr-4 py-4 rounded-xl text-lg focus:outline-none focus:ring-4 focus:ring-white/30">
                    </div>
                    <button class="w-full md:w-auto bg-gray-900 text-white px-8 py-4 rounded-xl font-semibold hover:bg-gray-800">Search</button>
                </div>
            </div>
        </div>

        <div class="max-w-7xl mx-auto px-4 py-12">
            <div class="flex overflow-x-auto gap-4 pb-4">
                <button onclick="filterCategory('all')" class="category-btn flex-shrink-0 px-6 py-3 bg-orange-500 text-white rounded-full font-medium">All Events</button>
                <button onclick="filterCategory('music')" class="category-btn flex-shrink-0 px-6 py-3 bg-gray-100 hover:bg-gray-200 rounded-full font-medium"><i class="fas fa-music mr-2"></i>Music</button>
                <button onclick="filterCategory('sport')" class="category-btn flex-shrink-0 px-6 py-3 bg-gray-100 hover:bg-gray-200 rounded-full font-medium"><i class="fas fa-futbol mr-2"></i>Sports</button>
                <button onclick="filterCategory('business')" class="category-btn flex-shrink-0 px-6 py-3 bg-gray-100 hover:bg-gray-200 rounded-full font-medium"><i class="fas fa-briefcase mr-2"></i>Business</button>
                <button onclick="filterCategory('nightlife')" class="category-btn flex-shrink-0 px-6 py-3 bg-gray-100 hover:bg-gray-200 rounded-full font-medium"><i class="fas fa-cocktail mr-2"></i>Nightlife</button>
            </div>
        </div>

        <div class="max-w-7xl mx-auto px-4 pb-12">
            <h2 class="text-2xl font-bold mb-6">Featured Events</h2>
            <div id="eventsGrid" class="grid md:grid-cols-2 lg:grid-cols-3 gap-6"></div>
        </div>

        <div class="max-w-7xl mx-auto px-4 py-12">
            <div class="flex justify-between items-center mb-6">
                <h2 class="text-2xl font-bold">Explore by Country</h2>
                <span class="text-sm text-gray-500">12 countries available</span>
            </div>
            <div id="countriesGrid" class="country-grid"></div>
        </div>
    </div>

    <!-- Promoter View -->
    <div id="view-promoter" class="view hidden pt-16">
        <div class="gradient-bg py-20">
            <div class="max-w-7xl mx-auto px-4">
                <div class="grid md:grid-cols-2 gap-12 items-center">
                    <div class="text-white">
                        <div class="inline-flex items-center bg-white/20 backdrop-blur rounded-full px-4 py-2 mb-6">
                            <i class="fas fa-passport mr-2"></i>
                            <span class="font-medium">The Universal Pass for African Events</span>
                        </div>
                        <h1 class="text-4xl md:text-5xl font-bold mb-6">Sell Tickets Across Africa, Get Paid Anywhere</h1>
                        <p class="text-xl mb-8 opacity-90">One platform, 12 countries, multiple currencies. Create events, reach fans, and receive payouts in your preferred currency.</p>
                        <div class="flex flex-wrap gap-4">
                            <button onclick="showView('dashboard')" class="bg-white text-orange-500 px-8 py-4 rounded-xl font-bold hover:bg-gray-100">Create Promoter Account</button>
                            <button class="border-2 border-white text-white px-8 py-4 rounded-xl font-bold hover:bg-white/10">Watch Demo</button>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <div class="max-w-7xl mx-auto px-4 py-20">
            <h2 class="text-3xl font-bold text-center mb-12">Everything You Need to Succeed</h2>
            <div class="grid md:grid-cols-3 gap-8">
                <div class="bg-white rounded-2xl p-8 shadow-lg card-hover">
                    <div class="w-14 h-14 bg-orange-100 rounded-xl flex items-center justify-center mb-6">
                        <i class="fas fa-globe-africa text-2xl text-orange-500"></i>
                    </div>
                    <h3 class="text-xl font-bold mb-3">Cross-Border Sales</h3>
                    <p class="text-gray-600">Sell tickets across 12 African countries from one dashboard.</p>
                </div>
                <div class="bg-white rounded-2xl p-8 shadow-lg card-hover">
                    <div class="w-14 h-14 bg-green-100 rounded-xl flex items-center justify-center mb-6">
                        <i class="fas fa-mobile-alt text-2xl text-green-500"></i>
                    </div>
                    <h3 class="text-xl font-bold mb-3">Local Payment Methods</h3>
                    <p class="text-gray-600">Accept cards, bank transfers, Orange Money, MTN MoMo, M-Pesa, and more.</p>
                </div>
                <div class="bg-white rounded-2xl p-8 shadow-lg card-hover">
                    <div class="w-14 h-14 bg-blue-100 rounded-xl flex items-center justify-center mb-6">
                        <i class="fas fa-chart-line text-2xl text-blue-500"></i>
                    </div>
                    <h3 class="text-xl font-bold mb-3">Real-Time Analytics</h3>
                    <p class="text-gray-600">Track sales by country, city, campaign. Optimize your marketing.</p>
                </div>
            </div>
        </div>
    </div>

    <!-- Pricing View -->
    <div id="view-pricing" class="view hidden pt-16">
        <div class="max-w-7xl mx-auto px-4 py-20">
            <div class="text-center mb-12">
                <div class="inline-flex items-center bg-orange-100 rounded-full px-4 py-2 mb-4">
                    <i class="fas fa-passport mr-2 text-orange-500"></i>
                    <span class="text-orange-600 font-medium">The Universal Pass for African Events</span>
                </div>
                <h1 class="text-4xl font-bold mb-4">Simple, Transparent Pricing</h1>
                <p class="text-gray-600">No monthly fees. Pay only when you sell.</p>
            </div>
            
            <div class="grid md:grid-cols-3 gap-8 max-w-5xl mx-auto">
                <div class="bg-white rounded-2xl p-8 shadow-lg border-2 border-gray-100">
                    <h3 class="text-xl font-bold mb-2">Starter</h3>
                    <p class="text-gray-600 mb-6">For small events</p>
                    <div class="mb-6">
                        <span class="text-4xl font-bold">5%</span>
                        <span class="text-gray-500"> per ticket</span>
                    </div>
                    <ul class="space-y-3 mb-8">
                        <li class="flex items-center"><i class="fas fa-check text-green-500 mr-3"></i>Unlimited events</li>
                        <li class="flex items-center"><i class="fas fa-check text-green-500 mr-3"></i>Basic analytics</li>
                        <li class="flex items-center"><i class="fas fa-check text-green-500 mr-3"></i>Email support</li>
                    </ul>
                    <button class="w-full py-3 border-2 border-orange-500 text-orange-500 rounded-xl font-semibold hover:bg-orange-50">Get Started</button>
                </div>
                
                <div class="bg-white rounded-2xl p-8 shadow-xl border-2 border-orange-500 relative">
                    <div class="absolute -top-4 left-1/2 -translate-x-1/2 gradient-bg text-white px-4 py-1 rounded-full text-sm font-medium">Most Popular</div>
                    <h3 class="text-xl font-bold mb-2">Professional</h3>
                    <p class="text-gray-600 mb-6">For growing organizations</p>
                    <div class="mb-6">
                        <span class="text-4xl font-bold">3.5%</span>
                        <span class="text-gray-500"> per ticket</span>
                    </div>
                    <ul class="space-y-3 mb-8">
                        <li class="flex items-center"><i class="fas fa-check text-green-500 mr-3"></i>Everything in Starter</li>
                        <li class="flex items-center"><i class="fas fa-check text-green-500 mr-3"></i>Advanced analytics</li>
                        <li class="flex items-center"><i class="fas fa-check text-green-500 mr-3"></i>Promo codes</li>
                        <li class="flex items-center"><i class="fas fa-check text-green-500 mr-3"></i>Priority support</li>
                    </ul>
                    <button onclick="showView('dashboard')" class="w-full py-3 gradient-bg text-white rounded-xl font-semibold hover:opacity-90">Get Started</button>
                </div>
                
                <div class="bg-white rounded-2xl p-8 shadow-lg border-2 border-gray-100">
                    <h3 class="text-xl font-bold mb-2">Enterprise</h3>
                    <p class="text-gray-600 mb-6">For large operations</p>
                    <div class="mb-6">
                        <span class="text-4xl font-bold">Custom</span>
                    </div>
                    <ul class="space-y-3 mb-8">
                        <li class="flex items-center"><i class="fas fa-check text-green-500 mr-3"></i>Everything in Pro</li>
                        <li class="flex items-center"><i class="fas fa-check text-green-500 mr-3"></i>Custom rates</li>
                        <li class="flex items-center"><i class="fas fa-check text-green-500 mr-3"></i>API access</li>
                        <li class="flex items-center"><i class="fas fa-check text-green-500 mr-3"></i>Dedicated manager</li>
                    </ul>
                    <button class="w-full py-3 border-2 border-gray-300 text-gray-700 rounded-xl font-semibold hover:bg-gray-50">Contact Sales</button>
                </div>
            </div>
        </div>
    </div>
        <!-- Dashboard View -->
    <div id="view-dashboard" class="view hidden pt-16">
        <div class="flex">
            <!-- Sidebar -->
            <div class="w-64 bg-white h-screen fixed left-0 top-16 shadow-lg overflow-y-auto">
                <div class="p-4">
                    <div class="bg-gray-100 rounded-xl p-4 mb-6">
                        <div class="flex items-center space-x-3">
                            <div class="w-12 h-12 gradient-bg rounded-xl flex items-center justify-center text-white font-bold text-lg">A</div>
                            <div>
                                <p class="font-semibold">Afrique Events</p>
                                <p class="text-sm text-gray-500">Professional Plan</p>
                            </div>
                        </div>
                    </div>
                    
                    <nav class="space-y-2">
                        <button onclick="showDashboardSection('overview')" class="sidebar-item active w-full flex items-center space-x-3 px-4 py-3 rounded-xl text-left">
                            <i class="fas fa-home w-5"></i><span>Overview</span>
                        </button>
                        <button onclick="showDashboardSection('events')" class="sidebar-item w-full flex items-center space-x-3 px-4 py-3 rounded-xl text-left text-gray-600 hover:bg-gray-100">
                            <i class="fas fa-calendar w-5"></i><span>Events</span>
                        </button>
                        <button onclick="showDashboardSection('analytics')" class="sidebar-item w-full flex items-center space-x-3 px-4 py-3 rounded-xl text-left text-gray-600 hover:bg-gray-100">
                            <i class="fas fa-chart-bar w-5"></i><span>Analytics</span>
                        </button>
                        <button onclick="showDashboardSection('marketing')" class="sidebar-item w-full flex items-center space-x-3 px-4 py-3 rounded-xl text-left text-gray-600 hover:bg-gray-100">
                            <i class="fas fa-bullhorn w-5"></i><span>Marketing</span>
                        </button>
                        <button onclick="showDashboardSection('payouts')" class="sidebar-item w-full flex items-center space-x-3 px-4 py-3 rounded-xl text-left text-gray-600 hover:bg-gray-100">
                            <i class="fas fa-wallet w-5"></i><span>Payouts</span>
                        </button>
                        <button onclick="showDashboardSection('team')" class="sidebar-item w-full flex items-center space-x-3 px-4 py-3 rounded-xl text-left text-gray-600 hover:bg-gray-100">
                            <i class="fas fa-users w-5"></i><span>Team</span>
                        </button>
                        <button onclick="showDashboardSection('settings')" class="sidebar-item w-full flex items-center space-x-3 px-4 py-3 rounded-xl text-left text-gray-600 hover:bg-gray-100">
                            <i class="fas fa-cog w-5"></i><span>Settings</span>
                        </button>
                        
                        <div class="border-t my-4 pt-4">
                            <p class="text-xs font-semibold text-gray-400 uppercase tracking-wider mb-3 px-4">Operations</p>
                            <button onclick="showDashboardSection('scanner')" class="sidebar-item w-full flex items-center space-x-3 px-4 py-3 rounded-xl text-left text-gray-600 hover:bg-gray-100">
                                <i class="fas fa-qrcode w-5"></i><span>Ticket Scanner</span>
                            </button>
                            <button onclick="showDashboardSection('email-templates')" class="sidebar-item w-full flex items-center space-x-3 px-4 py-3 rounded-xl text-left text-gray-600 hover:bg-gray-100">
                                <i class="fas fa-envelope w-5"></i><span>Email Templates</span>
                            </button>
                            <button onclick="showDashboardSection('sms-templates')" class="sidebar-item w-full flex items-center space-x-3 px-4 py-3 rounded-xl text-left text-gray-600 hover:bg-gray-100">
                                <i class="fas fa-sms w-5"></i><span>SMS Templates</span>
                            </button>
                        </div>
                        
                        <div class="border-t my-4 pt-4">
                            <p class="text-xs font-semibold text-gray-400 uppercase tracking-wider mb-3 px-4">Content Management</p>
                            <button onclick="showDashboardSection('cms-site')" class="sidebar-item w-full flex items-center space-x-3 px-4 py-3 rounded-xl text-left text-gray-600 hover:bg-gray-100">
                                <i class="fas fa-globe w-5"></i><span>Site Settings</span>
                            </button>
                            <button onclick="showDashboardSection('cms-faqs')" class="sidebar-item w-full flex items-center space-x-3 px-4 py-3 rounded-xl text-left text-gray-600 hover:bg-gray-100">
                                <i class="fas fa-question-circle w-5"></i><span>FAQs</span>
                            </button>
                            <button onclick="showDashboardSection('cms-countries')" class="sidebar-item w-full flex items-center space-x-3 px-4 py-3 rounded-xl text-left text-gray-600 hover:bg-gray-100">
                                <i class="fas fa-flag w-5"></i><span>Countries</span>
                            </button>
                        </div>
                    </nav>
                </div>
            </div>

            <!-- Main Content -->
            <div class="ml-64 flex-1 p-8">
                <!-- Overview Section -->
                <div id="section-overview" class="dashboard-section">
                    <h1 class="text-2xl font-bold mb-8">Welcome back!</h1>
                    <div class="grid grid-cols-4 gap-6 mb-8">
                        <div class="bg-white rounded-2xl p-6 shadow-sm">
                            <div class="w-12 h-12 bg-green-100 rounded-xl flex items-center justify-center mb-4">
                                <i class="fas fa-money-bill-wave text-green-500 text-xl"></i>
                            </div>
                            <p class="text-gray-600 text-sm">Total Revenue</p>
                            <p class="text-2xl font-bold">XOF 45,230,000</p>
                        </div>
                        <div class="bg-white rounded-2xl p-6 shadow-sm">
                            <div class="w-12 h-12 bg-blue-100 rounded-xl flex items-center justify-center mb-4">
                                <i class="fas fa-ticket-alt text-blue-500 text-xl"></i>
                            </div>
                            <p class="text-gray-600 text-sm">Tickets Sold</p>
                            <p class="text-2xl font-bold">3,847</p>
                        </div>
                        <div class="bg-white rounded-2xl p-6 shadow-sm">
                            <div class="w-12 h-12 bg-purple-100 rounded-xl flex items-center justify-center mb-4">
                                <i class="fas fa-calendar-check text-purple-500 text-xl"></i>
                            </div>
                            <p class="text-gray-600 text-sm">Active Events</p>
                            <p class="text-2xl font-bold">5</p>
                        </div>
                        <div class="bg-white rounded-2xl p-6 shadow-sm">
                            <div class="w-12 h-12 bg-orange-100 rounded-xl flex items-center justify-center mb-4">
                                <i class="fas fa-globe text-orange-500 text-xl"></i>
                            </div>
                            <p class="text-gray-600 text-sm">Countries</p>
                            <p class="text-2xl font-bold">6</p>
                        </div>
                    </div>
                </div>

                <!-- Events Section -->
                <div id="section-events" class="dashboard-section hidden">
                    <div class="flex justify-between items-center mb-8">
                        <h1 class="text-2xl font-bold">My Events</h1>
                        <button onclick="showDashboardSection('create-event')" class="gradient-bg text-white px-6 py-3 rounded-xl font-semibold">
                            <i class="fas fa-plus mr-2"></i>Create Event
                        </button>
                    </div>
                    <div class="bg-white rounded-2xl p-6 shadow-sm">
                        <div class="grid md:grid-cols-3 gap-6">
                            <div class="border rounded-xl p-4">
                                <h3 class="font-bold mb-2">Dakar Music Festival 2025</h3>
                                <p class="text-sm text-gray-500 mb-4">Mar 15, 2025 • Dakar</p>
                                <div class="flex justify-between text-sm">
                                    <span>2,340 / 5,000 sold</span>
                                    <span class="text-green-500">Live</span>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- Create Event Section -->
                <div id="section-create-event" class="dashboard-section hidden">
                    <button onclick="showDashboardSection('events')" class="text-gray-500 hover:text-gray-700 mb-4 flex items-center">
                        <i class="fas fa-arrow-left mr-2"></i>Back
                    </button>
                    <h1 class="text-2xl font-bold mb-8">Create New Event</h1>
                    <div class="bg-white rounded-2xl p-6 shadow-sm">
                        <div class="grid grid-cols-2 gap-4 mb-4">
                            <div>
                                <label class="block text-sm font-medium mb-2">Title (English)</label>
                                <input type="text" class="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-orange-500 focus:outline-none">
                            </div>
                            <div>
                                <label class="block text-sm font-medium mb-2">Title (Français)</label>
                                <input type="text" class="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-orange-500 focus:outline-none">
                            </div>
                        </div>
                        <div class="grid grid-cols-2 gap-4 mb-4">
                            <div>
                                <label class="block text-sm font-medium mb-2">Country</label>
                                <select id="eventCountrySelect" onchange="updateEventCurrency()" class="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-orange-500 focus:outline-none"></select>
                            </div>
                            <div>
                                <label class="block text-sm font-medium mb-2">Date & Time</label>
                                <input type="datetime-local" class="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-orange-500 focus:outline-none">
                            </div>
                        </div>
                        <h3 class="font-bold mb-4 mt-6">Ticket Types</h3>
                        <div class="border rounded-xl p-4 mb-4">
                            <div class="grid grid-cols-4 gap-4">
                                <div>
                                    <label class="block text-sm font-medium mb-2">Name</label>
                                    <input type="text" value="Regular" class="w-full px-3 py-2 border rounded-lg">
                                </div>
                                <div>
                                    <label class="block text-sm font-medium mb-2">Price</label>
                                    <div class="flex">
                                        <span id="ticketCurrency1" class="bg-gray-100 px-3 py-2 border border-r-0 rounded-l-lg">XOF</span>
                                        <input type="number" value="5000" class="w-full px-3 py-2 border rounded-r-lg">
                                    </div>
                                </div>
                                <div>
                                    <label class="block text-sm font-medium mb-2">Quantity</label>
                                    <input type="number" value="500" class="w-full px-3 py-2 border rounded-lg">
                                </div>
                            </div>
                        </div>
                        <button onclick="showToast('Event published successfully!', 'success')" class="gradient-bg text-white px-8 py-3 rounded-xl font-semibold">Publish Event</button>
                    </div>
                </div>

                <!-- Analytics Section -->
                <div id="section-analytics" class="dashboard-section hidden">
                    <h1 class="text-2xl font-bold mb-8">Analytics</h1>
                    <div class="grid grid-cols-2 gap-6">
                        <div class="bg-white rounded-2xl p-6 shadow-sm">
                            <h3 class="font-bold mb-4">Sales by Channel</h3>
                            <div class="space-y-3">
                                <div class="flex justify-between"><span>Direct</span><span class="font-medium">58%</span></div>
                                <div class="flex justify-between"><span>Social Media</span><span class="font-medium">24%</span></div>
                                <div class="flex justify-between"><span>Affiliates</span><span class="font-medium">12%</span></div>
                                <div class="flex justify-between"><span>Email</span><span class="font-medium">6%</span></div>
                            </div>
                        </div>
                        <div class="bg-white rounded-2xl p-6 shadow-sm">
                            <h3 class="font-bold mb-4">Payment Methods</h3>
                            <div class="space-y-3">
                                <div class="flex justify-between"><span>Orange Money</span><span class="font-medium">45%</span></div>
                                <div class="flex justify-between"><span>MTN MoMo</span><span class="font-medium">30%</span></div>
                                <div class="flex justify-between"><span>Cards</span><span class="font-medium">20%</span></div>
                                <div class="flex justify-between"><span>Bank Transfer</span><span class="font-medium">5%</span></div>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- Marketing Section with Referral Program -->
                <div id="section-marketing" class="dashboard-section hidden">
                    <h1 class="text-2xl font-bold mb-8">Marketing Tools</h1>
                    <div class="grid grid-cols-2 gap-6 mb-6">
                        <div class="bg-white rounded-2xl p-6 shadow-sm">
                            <h3 class="font-bold mb-4">Promo Codes</h3>
                            <div class="p-4 bg-gray-50 rounded-xl mb-3">
                                <p class="font-mono font-bold">DAKAR20</p>
                                <p class="text-sm text-gray-500">20% off • 127 uses</p>
                            </div>
                        </div>
                        <div class="bg-white rounded-2xl p-6 shadow-sm">
                            <h3 class="font-bold mb-4">Affiliate Links</h3>
                            <div class="p-4 bg-gray-50 rounded-xl">
                                <p class="font-bold">@InfluencerSN</p>
                                <p class="text-sm text-gray-500">47 sales • XOF 234,500</p>
                            </div>
                        </div>
                    </div>
                    
                    <!-- Referral Program -->
                    <div class="bg-gradient-to-r from-orange-50 to-yellow-50 rounded-2xl p-6 border-2 border-orange-200">
                        <div class="flex items-center justify-between mb-6">
                            <div>
                                <h3 class="text-xl font-bold mb-2">Referral Program</h3>
                                <p class="text-gray-600">Earn 10% commission on every promoter you invite</p>
                            </div>
                            <div class="text-right">
                                <p class="text-sm text-gray-600">Total Earned</p>
                                <p class="text-2xl font-bold text-orange-600">XOF 1,250,000</p>
                            </div>
                        </div>
                        <div class="grid grid-cols-3 gap-4 mb-6">
                            <div class="bg-white rounded-xl p-4 text-center">
                                <p class="text-2xl font-bold text-orange-500">12</p>
                                <p class="text-sm text-gray-600">Referrals</p>
                            </div>
                            <div class="bg-white rounded-xl p-4 text-center">
                                <p class="text-2xl font-bold text-green-600">8</p>
                                <p class="text-sm text-gray-600">Active</p>
                            </div>
                            <div class="bg-white rounded-xl p-4 text-center">
                                <p class="text-2xl font-bold text-blue-600">XOF 150K</p>
                                <p class="text-sm text-gray-600">Avg/Referral</p>
                            </div>
                        </div>
                        <div class="bg-white rounded-xl p-4">
                            <label class="block text-sm font-medium mb-2">Your Referral Link</label>
                            <div class="flex gap-2">
                                <input type="text" value="https://passafrik.com/ref/AFE2025" readonly class="flex-1 px-4 py-3 border rounded-lg bg-gray-50 font-mono text-sm">
                                <button onclick="copyReferralLink()" class="px-6 py-3 gradient-bg text-white rounded-lg font-medium">
                                    <i class="fas fa-copy mr-2"></i>Copy
                                </button>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- Payouts Section -->
                <div id="section-payouts" class="dashboard-section hidden">
                    <h1 class="text-2xl font-bold mb-8">Payouts & Finance</h1>
                    <div class="grid grid-cols-3 gap-6 mb-8">
                        <div class="bg-white rounded-2xl p-6 shadow-sm">
                            <p class="text-sm text-gray-600 mb-2">Available Balance</p>
                            <p class="text-3xl font-bold text-green-600">XOF 12,450,000</p>
                        </div>
                        <div class="bg-white rounded-2xl p-6 shadow-sm">
                            <p class="text-sm text-gray-600 mb-2">Pending (Escrow)</p>
                            <p class="text-3xl font-bold text-orange-500">XOF 8,230,000</p>
                        </div>
                        <div class="bg-white rounded-2xl p-6 shadow-sm">
                            <p class="text-sm text-gray-600 mb-2">Total Paid Out</p>
                            <p class="text-3xl font-bold">XOF 45,670,000</p>
                        </div>
                    </div>
                </div>

                <!-- Team Section -->
                <div id="section-team" class="dashboard-section hidden">
                    <div class="flex justify-between items-center mb-8">
                        <h1 class="text-2xl font-bold">Team Management</h1>
                        <button onclick="showInviteMemberModal()" class="gradient-bg text-white px-6 py-3 rounded-xl font-semibold">
                            <i class="fas fa-plus mr-2"></i>Invite Member
                        </button>
                    </div>
                    <div class="bg-white rounded-2xl p-6 shadow-sm">
                        <div id="teamMembersList" class="space-y-4">
                            <div class="flex items-center justify-between p-4 bg-gray-50 rounded-xl">
                                <div class="flex items-center">
                                    <div class="w-12 h-12 gradient-bg rounded-full flex items-center justify-center text-white font-bold mr-4">AM</div>
                                    <div>
                                        <p class="font-bold">Amadou Mbaye</p>
                                        <p class="text-sm text-gray-500">amadou@afrique-events.com</p>
                                    </div>
                                </div>
                                <div class="flex items-center space-x-3">
                                    <span class="bg-purple-100 text-purple-600 px-3 py-1 rounded-full text-sm font-medium">Owner</span>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- Settings Section -->
                <div id="section-settings" class="dashboard-section hidden">
                    <h1 class="text-2xl font-bold mb-8">Settings</h1>
                    <div class="bg-white rounded-2xl p-6 shadow-sm">
                        <h3 class="font-bold mb-6">Organization Info</h3>
                        <div class="space-y-4">
                            <div>
                                <label class="block text-sm font-medium mb-2">Organization Name</label>
                                <input type="text" value="Afrique Events" class="w-full px-4 py-3 border rounded-xl">
                            </div>
                            <button onclick="showToast('Settings saved!', 'success')" class="gradient-bg text-white px-6 py-3 rounded-xl font-semibold">Save Changes</button>
                        </div>
                    </div>
                </div>
                                <!-- Scanner Section -->
                <div id="section-scanner" class="dashboard-section hidden">
                    <h1 class="text-2xl font-bold mb-8">Ticket Scanner</h1>
                    <div class="bg-white rounded-2xl p-6 shadow-sm">
                        <div class="mb-6">
                            <label class="block text-sm font-medium mb-2">Select Event</label>
                            <select class="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-orange-500 focus:outline-none">
                                <option>Dakar Music Festival 2025 - Mar 15, 2025</option>
                                <option>Lagos Afrobeats Night - Feb 28, 2025</option>
                            </select>
                        </div>
                        <div class="bg-gray-900 rounded-2xl h-64 flex items-center justify-center mb-6">
                            <div class="text-center text-white">
                                <i class="fas fa-qrcode text-6xl mb-4 opacity-50"></i>
                                <p class="opacity-75">QR Scanner View</p>
                                <p class="text-sm opacity-50">Camera access required for live scanning</p>
                            </div>
                        </div>
                        <div class="grid grid-cols-4 gap-4 mb-6">
                            <div class="bg-green-50 rounded-xl p-4 text-center">
                                <p class="text-green-600 text-sm">Checked In</p>
                                <p class="text-2xl font-bold text-green-600">247</p>
                            </div>
                            <div class="bg-blue-50 rounded-xl p-4 text-center">
                                <p class="text-blue-600 text-sm">Total</p>
                                <p class="text-2xl font-bold text-blue-600">500</p>
                            </div>
                            <div class="bg-red-50 rounded-xl p-4 text-center">
                                <p class="text-red-600 text-sm">Rejected</p>
                                <p class="text-2xl font-bold text-red-600">3</p>
                            </div>
                            <div class="bg-purple-50 rounded-xl p-4 text-center">
                                <p class="text-purple-600 text-sm">VIP</p>
                                <p class="text-2xl font-bold text-purple-600">89</p>
                            </div>
                        </div>
                        <div class="border-t pt-6">
                            <h4 class="font-bold mb-4">Manual Entry</h4>
                            <div class="flex gap-3">
                                <input type="text" placeholder="Enter ticket code: PA-1-XXXXX" class="flex-1 px-4 py-3 border rounded-xl font-mono">
                                <button onclick="showToast('Ticket validated!', 'success')" class="px-6 py-3 gradient-bg text-white rounded-xl font-semibold">Validate</button>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- Email Templates Section -->
                <div id="section-email-templates" class="dashboard-section hidden">
                    <h1 class="text-2xl font-bold mb-8">Email Templates</h1>
                    <div class="bg-white rounded-2xl p-6 shadow-sm">
                        <div class="grid grid-cols-4 gap-6">
                            <div class="border-r pr-6">
                                <h3 class="font-bold mb-4">Templates</h3>
                                <div class="space-y-2">
                                    <div class="p-3 bg-orange-50 border-2 border-orange-500 rounded-xl cursor-pointer">
                                        <p class="font-medium text-sm">Order Confirmation</p>
                                        <p class="text-xs text-gray-500">Active</p>
                                    </div>
                                    <div class="p-3 border rounded-xl cursor-pointer hover:bg-gray-50">
                                        <p class="font-medium text-sm">Ticket Delivery</p>
                                        <p class="text-xs text-gray-500">Active</p>
                                    </div>
                                    <div class="p-3 border rounded-xl cursor-pointer hover:bg-gray-50">
                                        <p class="font-medium text-sm">Event Reminder</p>
                                        <p class="text-xs text-gray-500">Active</p>
                                    </div>
                                    <div class="p-3 border rounded-xl cursor-pointer hover:bg-gray-50">
                                        <p class="font-medium text-sm">Payment Receipt</p>
                                        <p class="text-xs text-gray-500">Active</p>
                                    </div>
                                    <div class="p-3 border rounded-xl cursor-pointer hover:bg-gray-50">
                                        <p class="font-medium text-sm">Refund Confirmation</p>
                                        <p class="text-xs text-gray-500">Active</p>
                                    </div>
                                    <div class="p-3 border rounded-xl cursor-pointer hover:bg-gray-50">
                                        <p class="font-medium text-sm">Event Cancelled</p>
                                        <p class="text-xs text-gray-500">Draft</p>
                                    </div>
                                    <div class="p-3 border rounded-xl cursor-pointer hover:bg-gray-50">
                                        <p class="font-medium text-sm">Welcome Email</p>
                                        <p class="text-xs text-gray-500">Active</p>
                                    </div>
                                    <div class="p-3 border rounded-xl cursor-pointer hover:bg-gray-50">
                                        <p class="font-medium text-sm">Password Reset</p>
                                        <p class="text-xs text-gray-500">Active</p>
                                    </div>
                                </div>
                            </div>
                            <div class="col-span-3">
                                <div class="flex justify-between items-center mb-6">
                                    <h3 class="font-bold">Edit: Order Confirmation</h3>
                                    <div class="flex gap-2">
                                        <button class="px-4 py-2 bg-gray-100 rounded-lg text-sm font-medium">EN</button>
                                        <button class="px-4 py-2 bg-orange-500 text-white rounded-lg text-sm font-medium">FR</button>
                                    </div>
                                </div>
                                <div class="mb-4">
                                    <label class="block text-sm font-medium mb-2">Subject Line</label>
                                    <input type="text" value="Vos billets pour {{event_name}} sont confirmés! 🎉" class="w-full px-4 py-3 border rounded-xl">
                                </div>
                                <div class="mb-4">
                                    <label class="block text-sm font-medium mb-2">Email Body</label>
                                    <textarea rows="10" class="w-full px-4 py-3 border rounded-xl font-mono text-sm">Bonjour {{customer_name}},

Merci pour votre achat! Vos billets pour {{event_name}} sont confirmés.

📅 Date: {{event_date}}
📍 Lieu: {{event_venue}}
🎫 Billets: {{ticket_count}} x {{ticket_type}}
💰 Total: {{order_total}}

Code(s) de billet:
{{ticket_codes}}

Présentez le code QR à l'entrée.

À bientôt!
L'équipe PassAfrik</textarea>
                                </div>
                                <div class="bg-orange-50 rounded-xl p-4 mb-4">
                                    <p class="text-sm font-medium mb-2">Variables disponibles (cliquez pour copier):</p>
                                    <div class="flex flex-wrap gap-2">
                                        <code class="bg-white px-2 py-1 rounded text-xs cursor-pointer hover:bg-orange-100">{{customer_name}}</code>
                                        <code class="bg-white px-2 py-1 rounded text-xs cursor-pointer hover:bg-orange-100">{{event_name}}</code>
                                        <code class="bg-white px-2 py-1 rounded text-xs cursor-pointer hover:bg-orange-100">{{event_date}}</code>
                                        <code class="bg-white px-2 py-1 rounded text-xs cursor-pointer hover:bg-orange-100">{{event_venue}}</code>
                                        <code class="bg-white px-2 py-1 rounded text-xs cursor-pointer hover:bg-orange-100">{{ticket_codes}}</code>
                                        <code class="bg-white px-2 py-1 rounded text-xs cursor-pointer hover:bg-orange-100">{{order_total}}</code>
                                    </div>
                                </div>
                                <div class="flex gap-3">
                                    <button class="px-6 py-3 bg-blue-500 text-white rounded-xl font-medium">
                                        <i class="fas fa-paper-plane mr-2"></i>Send Test
                                    </button>
                                    <button onclick="showToast('Template saved!', 'success')" class="px-6 py-3 gradient-bg text-white rounded-xl font-semibold">
                                        <i class="fas fa-save mr-2"></i>Save Template
                                    </button>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- SMS Templates Section -->
                <div id="section-sms-templates" class="dashboard-section hidden">
                    <h1 class="text-2xl font-bold mb-8">SMS Templates</h1>
                    <div class="grid lg:grid-cols-2 gap-6">
                        <div class="bg-white rounded-2xl p-6 shadow-sm">
                            <div class="flex justify-between items-center mb-6">
                                <h3 class="font-bold">Edit SMS Template</h3>
                                <div class="flex gap-2">
                                    <button class="px-3 py-1 bg-orange-500 text-white rounded-lg text-sm font-medium">EN</button>
                                    <button class="px-3 py-1 bg-gray-100 rounded-lg text-sm font-medium">FR</button>
                                </div>
                            </div>
                            <div class="mb-4">
                                <label class="block text-sm font-medium mb-2">Template Type</label>
                                <select class="w-full px-4 py-3 border rounded-xl">
                                    <option>Order Confirmation SMS</option>
                                    <option>Ticket Code SMS</option>
                                    <option>Event Reminder SMS</option>
                                    <option>Refund SMS</option>
                                </select>
                            </div>
                            <div class="mb-4">
                                <label class="block text-sm font-medium mb-2">SMS Message</label>
                                <textarea id="smsBody" rows="4" maxlength="160" oninput="updateSMSCount()" class="w-full px-4 py-3 border rounded-xl">PassAfrik: Your tickets for {{event_name}} confirmed! Code: {{ticket_code}}. Show at entry. {{event_date}}</textarea>
                                <div class="flex justify-between mt-2 text-sm">
                                    <span id="smsCharCount" class="text-gray-500">104/160 characters</span>
                                    <span id="smsSegments" class="text-gray-500">1 SMS segment</span>
                                </div>
                            </div>
                            <div class="bg-blue-50 rounded-xl p-4 mb-4">
                                <p class="text-sm font-medium mb-2">SMS Provider</p>
                                <div class="flex gap-2">
                                    <label class="flex items-center px-3 py-2 bg-white rounded-lg border cursor-pointer">
                                        <input type="radio" name="smsProvider" checked class="mr-2">
                                        <span class="text-sm">Twilio</span>
                                    </label>
                                    <label class="flex items-center px-3 py-2 bg-white rounded-lg border cursor-pointer">
                                        <input type="radio" name="smsProvider" class="mr-2">
                                        <span class="text-sm">Africa's Talking</span>
                                    </label>
                                </div>
                            </div>
                            <div class="flex gap-3">
                                <button class="px-6 py-3 bg-blue-500 text-white rounded-xl font-medium">
                                    <i class="fas fa-paper-plane mr-2"></i>Send Test
                                </button>
                                <button onclick="showToast('SMS template saved!', 'success')" class="px-6 py-3 gradient-bg text-white rounded-xl font-semibold">
                                    <i class="fas fa-save mr-2"></i>Save
                                </button>
                            </div>
                        </div>
                        <div class="bg-gray-100 rounded-2xl p-6 flex items-center justify-center">
                            <div class="w-72 bg-white rounded-3xl shadow-2xl p-4" style="height: 600px;">
                                <div class="bg-gray-100 rounded-2xl h-full p-4 overflow-hidden">
                                    <p class="text-xs text-gray-400 mb-4">Preview</p>
                                    <div class="bg-orange-100 rounded-2xl rounded-tl-none p-3 mb-2">
                                        <p class="text-sm" id="smsPreview">PassAfrik: Your tickets for Dakar Music Festival confirmed! Code: PA-1-ABC123. Show at entry. Mar 15, 2025</p>
                                    </div>
                                    <p class="text-xs text-gray-400">Today, 10:30 AM</p>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- CMS: Site Settings -->
                <div id="section-cms-site" class="dashboard-section hidden">
                    <h1 class="text-2xl font-bold mb-8">Site Settings</h1>
                    <div class="bg-white rounded-2xl p-6 shadow-sm">
                        <h3 class="font-bold mb-6">Branding</h3>
                        <div class="space-y-4">
                            <div>
                                <label class="block text-sm font-medium mb-2">Platform Name</label>
                                <input type="text" value="PassAfrik" class="w-full px-4 py-3 border rounded-xl">
                            </div>
                            <div>
                                <label class="block text-sm font-medium mb-2">Slogan (English)</label>
                                <input type="text" value="The Universal Pass for African Events" class="w-full px-4 py-3 border rounded-xl">
                            </div>
                            <div>
                                <label class="block text-sm font-medium mb-2">Slogan (Français)</label>
                                <input type="text" value="Le Pass Universel pour les Événements Africains" class="w-full px-4 py-3 border rounded-xl">
                            </div>
                            <div>
                                <label class="block text-sm font-medium mb-2">Support Email</label>
                                <input type="email" value="support@passafrik.com" class="w-full px-4 py-3 border rounded-xl">
                            </div>
                            <button onclick="showToast('Settings saved!', 'success')" class="gradient-bg text-white px-6 py-3 rounded-xl font-semibold">Save Changes</button>
                        </div>
                    </div>
                </div>

                <!-- CMS: FAQs -->
                <div id="section-cms-faqs" class="dashboard-section hidden">
                    <div class="flex justify-between items-center mb-8">
                        <h1 class="text-2xl font-bold">FAQ Management</h1>
                        <button onclick="showToast('FAQ added!', 'success')" class="gradient-bg text-white px-6 py-3 rounded-xl font-semibold">
                            <i class="fas fa-plus mr-2"></i>Add FAQ
                        </button>
                    </div>
                    <div class="bg-white rounded-2xl p-6 shadow-sm">
                        <table class="w-full">
                            <thead class="border-b">
                                <tr class="text-left text-sm">
                                    <th class="pb-4">Question</th>
                                    <th class="pb-4">Category</th>
                                    <th class="pb-4">Status</th>
                                    <th class="pb-4">Actions</th>
                                </tr>
                            </thead>
                            <tbody>
                                <tr class="border-t">
                                    <td class="py-4">How do I purchase tickets?</td>
                                    <td class="py-4"><span class="px-3 py-1 bg-blue-100 text-blue-600 rounded-full text-xs">Buying</span></td>
                                    <td class="py-4"><span class="px-3 py-1 bg-green-100 text-green-600 rounded-full text-xs">Active</span></td>
                                    <td class="py-4">
                                        <button class="text-blue-500 mr-3"><i class="fas fa-edit"></i></button>
                                        <button class="text-red-500"><i class="fas fa-trash"></i></button>
                                    </td>
                                </tr>
                                <tr class="border-t">
                                    <td class="py-4">What payment methods are accepted?</td>
                                    <td class="py-4"><span class="px-3 py-1 bg-blue-100 text-blue-600 rounded-full text-xs">Buying</span></td>
                                    <td class="py-4"><span class="px-3 py-1 bg-green-100 text-green-600 rounded-full text-xs">Active</span></td>
                                    <td class="py-4">
                                        <button class="text-blue-500 mr-3"><i class="fas fa-edit"></i></button>
                                        <button class="text-red-500"><i class="fas fa-trash"></i></button>
                                    </td>
                                </tr>
                            </tbody>
                        </table>
                    </div>
                </div>

                <!-- CMS: Countries -->
                <div id="section-cms-countries" class="dashboard-section hidden">
                    <div class="flex justify-between items-center mb-8">
                        <h1 class="text-2xl font-bold">Countries Management</h1>
                        <button onclick="showToast('Country configuration updated!', 'info')" class="gradient-bg text-white px-6 py-3 rounded-xl font-semibold">
                            <i class="fas fa-plus mr-2"></i>Add Country
                        </button>
                    </div>
                    <div class="grid grid-cols-3 gap-4">
                        <div class="bg-white rounded-xl p-6 shadow-sm">
                            <div class="flex items-center justify-between mb-4">
                                <span class="text-4xl">🇸🇳</span>
                                <button class="text-blue-500"><i class="fas fa-edit"></i></button>
                            </div>
                            <h3 class="font-bold">Sénégal</h3>
                            <p class="text-sm text-gray-500">Currency: XOF</p>
                            <p class="text-sm text-gray-500">Language: French</p>
                        </div>
                        <div class="bg-white rounded-xl p-6 shadow-sm">
                            <div class="flex items-center justify-between mb-4">
                                <span class="text-4xl">🇳🇬</span>
                                <button class="text-blue-500"><i class="fas fa-edit"></i></button>
                            </div>
                            <h3 class="font-bold">Nigeria</h3>
                            <p class="text-sm text-gray-500">Currency: NGN</p>
                            <p class="text-sm text-gray-500">Language: English</p>
                        </div>
                        <div class="bg-white rounded-xl p-6 shadow-sm">
                            <div class="flex items-center justify-between mb-4">
                                <span class="text-4xl">🇬🇭</span>
                                <button class="text-blue-500"><i class="fas fa-edit"></i></button>
                            </div>
                            <h3 class="font-bold">Ghana</h3>
                            <p class="text-sm text-gray-500">Currency: GHS</p>
                            <p class="text-sm text-gray-500">Language: English</p>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
        <!-- Invite Member Modal -->
    <div id="inviteMemberModal" class="modal-overlay fixed inset-0 bg-black/50 z-[80] hidden items-center justify-center p-4">
        <div class="modal-content bg-white rounded-2xl max-w-md w-full p-6">
            <div class="flex justify-between items-center mb-6">
                <h2 class="text-xl font-bold">Invite Team Member</h2>
                <button onclick="closeInviteMemberModal()" class="text-gray-400 hover:text-gray-600">
                    <i class="fas fa-times text-xl"></i>
                </button>
            </div>
            <form onsubmit="sendInvite(event)">
                <div class="mb-4">
                    <label class="block text-sm font-medium mb-2">Email Address</label>
                    <input type="email" id="inviteEmail" required class="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-orange-500 focus:outline-none" placeholder="colleague@example.com">
                </div>
                <div class="mb-6">
                    <label class="block text-sm font-medium mb-2">Role</label>
                    <select id="inviteRole" class="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-orange-500 focus:outline-none">
                        <option value="manager">Manager - Full access except billing</option>
                        <option value="cashier">Cashier - Sales and check-in only</option>
                        <option value="scanner">Scanner - Check-in only</option>
                    </select>
                </div>
                <button type="submit" class="w-full gradient-bg text-white py-3 rounded-xl font-semibold">Send Invitation</button>
            </form>
        </div>
    </div>

    <!-- Footer -->
    <footer class="bg-gray-900 text-white py-12 mt-12">
        <div class="max-w-7xl mx-auto px-4">
            <div class="grid md:grid-cols-4 gap-8">
                <div>
                    <div class="flex items-center mb-4">
                        <div class="w-10 h-10 gradient-bg rounded-xl flex items-center justify-center">
                            <i class="fas fa-passport text-white text-lg"></i>
                        </div>
                        <span class="ml-2 text-xl font-bold">PassAfrik</span>
                    </div>
                    <p class="text-gray-400 mb-2">The Universal Pass for African Events</p>
                    <p class="text-gray-500 text-sm">Pan-African ticketing platform</p>
                </div>
                <div>
                    <h4 class="font-bold mb-4">Platform</h4>
                    <ul class="space-y-2 text-gray-400">
                        <li><a href="#" onclick="showView('discover'); return false;" class="hover:text-white">Discover Events</a></li>
                        <li><a href="#" onclick="showView('promoter'); return false;" class="hover:text-white">For Promoters</a></li>
                        <li><a href="#" onclick="showView('pricing'); return false;" class="hover:text-white">Pricing</a></li>
                    </ul>
                </div>
                <div>
                    <h4 class="font-bold mb-4">Support</h4>
                    <ul class="space-y-2 text-gray-400">
                        <li><a href="#" class="hover:text-white">Help Center</a></li>
                        <li><a href="#" class="hover:text-white">Contact Us</a></li>
                        <li><a href="#" class="hover:text-white">API Docs</a></li>
                    </ul>
                </div>
                <div>
                    <h4 class="font-bold mb-4">Countries</h4>
                    <div id="footerCountries" class="flex flex-wrap gap-2"></div>
                </div>
            </div>
            <div class="border-t border-gray-800 mt-8 pt-8 text-center text-gray-400">
                <p>© 2025 PassAfrik. All rights reserved.</p>
            </div>
        </div>
    </footer>

    <script>
        // Global State
        let currentLang = 'en';
        let currentCountry = 'SN';
        let currentView = 'discover';
        let teamMembers = [
            { id: 1, name: 'Amadou Mbaye', email: 'amadou@afrique-events.com', role: 'owner', initials: 'AM' }
        ];

        // Data
        const countries = {
            SN: { name: 'Sénégal', currency: 'XOF', flag: '🇸🇳', language: 'fr', region: 'west' },
            CI: { name: "Côte d'Ivoire", currency: 'XOF', flag: '🇨🇮', language: 'fr', region: 'west' },
            GN: { name: 'Guinée', currency: 'GNF', flag: '🇬🇳', language: 'fr', region: 'west' },
            ML: { name: 'Mali', currency: 'XOF', flag: '🇲🇱', language: 'fr', region: 'west' },
            NG: { name: 'Nigeria', currency: 'NGN', flag: '🇳🇬', language: 'en', region: 'west' },
            GH: { name: 'Ghana', currency: 'GHS', flag: '🇬🇭', language: 'en', region: 'west' },
            LR: { name: 'Liberia', currency: 'LRD', flag: '🇱🇷', language: 'en', region: 'west' },
            SL: { name: 'Sierra Leone', currency: 'SLL', flag: '🇸🇱', language: 'en', region: 'west' },
            GM: { name: 'Gambia', currency: 'GMD', flag: '🇬🇲', language: 'en', region: 'west' },
            CD: { name: 'RD Congo', currency: 'CDF', flag: '🇨🇩', language: 'fr', region: 'central' },
            AO: { name: 'Angola', currency: 'AOA', flag: '🇦🇴', language: 'pt', region: 'southern' },
            KE: { name: 'Kenya', currency: 'KES', flag: '🇰🇪', language: 'en', region: 'east' }
        };

        const events = [
            { id: 1, title_en: 'Dakar Music Festival 2025', title_fr: 'Festival de Musique de Dakar 2025', category: 'music', country: 'SN', city: 'Dakar', date: '2025-03-15', price: 15000, currency: 'XOF', image: 'https://images.unsplash.com/photo-1459749411175-04bf5292ceea?w=800' },
            { id: 2, title_en: 'Lagos Afrobeats Night', title_fr: 'Soirée Afrobeats Lagos', category: 'music', country: 'NG', city: 'Lagos', date: '2025-02-28', price: 25000, currency: 'NGN', image: 'https://images.unsplash.com/photo-1514525253161-7a46d19cd819?w=800' },
            { id: 3, title_en: 'Accra Football Championship', title_fr: 'Championnat de Football Accra', category: 'sport', country: 'GH', city: 'Accra', date: '2025-05-10', price: 150, currency: 'GHS', image: 'https://images.unsplash.com/photo-1574629810360-7efbbe195018?w=800' },
            { id: 4, title_en: 'Abidjan Business Summit', title_fr: "Sommet des Affaires d'Abidjan", category: 'business', country: 'CI', city: 'Abidjan', date: '2025-04-20', price: 50000, currency: 'XOF', image: 'https://images.unsplash.com/photo-1540575467063-178a50c2df87?w=800' }
        ];

        // Functions
        function renderCountrySelectors() {
            const countrySelect = document.getElementById('countrySelect');
            const eventCountrySelect = document.getElementById('eventCountrySelect');
            const countriesGrid = document.getElementById('countriesGrid');
            const footerCountries = document.getElementById('footerCountries');
            
            if (countrySelect) {
                countrySelect.innerHTML = Object.entries(countries).map(([code, country]) => 
                    `<option value="${code}">${country.flag} ${country.name}</option>`
                ).join('');
                countrySelect.value = currentCountry;
            }
            
            if (eventCountrySelect) {
                eventCountrySelect.innerHTML = Object.entries(countries).map(([code, country]) => 
                    `<option value="${code}">${country.flag} ${country.name}</option>`
                ).join('');
            }
            
            if (countriesGrid) {
                countriesGrid.innerHTML = Object.entries(countries).map(([code, country]) => `
                    <div class="bg-white rounded-xl p-4 text-center cursor-pointer card-hover shadow-sm hover:shadow-lg">
                        <span class="text-3xl">${country.flag}</span>
                        <p class="font-semibold mt-2 text-sm">${country.name}</p>
                        <p class="text-xs text-gray-500">${country.currency}</p>
                    </div>
                `).join('');
            }
            
            if (footerCountries) {
                footerCountries.innerHTML = Object.entries(countries).map(([code, country]) => 
                    `<span class="text-2xl cursor-pointer hover:scale-110 transition-transform" title="${country.name}">${country.flag}</span>`
                ).join('');
            }
        }

        function setLanguage(lang) {
            currentLang = lang;
            document.getElementById('lang-en').className = lang === 'en' ? 'px-3 py-1 rounded-md text-sm font-medium bg-orange-500 text-white' : 'px-3 py-1 rounded-md text-sm font-medium text-gray-600';
            document.getElementById('lang-fr').className = lang === 'fr' ? 'px-3 py-1 rounded-md text-sm font-medium bg-orange-500 text-white' : 'px-3 py-1 rounded-md text-sm font-medium text-gray-600';
            renderEvents();
        }

        function changeCountry(code) {
            currentCountry = code;
            renderEvents();
        }

        function showView(view) {
            document.querySelectorAll('.view').forEach(v => v.classList.add('hidden'));
            const viewEl = document.getElementById(`view-${view}`);
            if (viewEl) viewEl.classList.remove('hidden');
            currentView = view;
            if (view === 'dashboard') showDashboardSection('overview');
            window.scrollTo(0, 0);
        }

        function showDashboardSection(section) {
            document.querySelectorAll('.dashboard-section').forEach(s => s.classList.add('hidden'));
            const sectionEl = document.getElementById(`section-${section}`);
            if (sectionEl) sectionEl.classList.remove('hidden');
            
            document.querySelectorAll('.sidebar-item').forEach(item => {
                item.classList.remove('active');
                item.classList.add('text-gray-600', 'hover:bg-gray-100');
            });
            
            const activeBtn = document.querySelector(`button[onclick*="showDashboardSection('${section}')"]`);
            if (activeBtn && activeBtn.classList.contains('sidebar-item')) {
                activeBtn.classList.add('active');
                activeBtn.classList.remove('text-gray-600', 'hover:bg-gray-100');
            }
        }

        function filterCategory(category) {
            document.querySelectorAll('.category-btn').forEach(btn => {
                btn.classList.remove('bg-orange-500', 'text-white');
                btn.classList.add('bg-gray-100');
            });
            event.target.closest('.category-btn').classList.add('bg-orange-500', 'text-white');
            event.target.closest('.category-btn').classList.remove('bg-gray-100');
            renderEvents(category);
        }

        function formatPrice(price, currency) {
            return `${currency} ${price.toLocaleString()}`;
        }

        function renderEvents(categoryFilter = 'all') {
            const grid = document.getElementById('eventsGrid');
            if (!grid) return;
            
            let filtered = categoryFilter === 'all' ? events : events.filter(e => e.category === categoryFilter);
            
            grid.innerHTML = filtered.map(event => `
                <div class="bg-white rounded-2xl overflow-hidden shadow-sm card-hover cursor-pointer">
                    <div class="relative">
                        <img src="${event.image}" alt="${currentLang === 'en' ? event.title_en : event.title_fr}" class="w-full h-48 object-cover">
                        <div class="absolute top-4 left-4 bg-white/90 backdrop-blur px-3 py-1 rounded-full text-sm font-medium">
                            ${countries[event.country].flag} ${countries[event.country].name}
                        </div>
                    </div>
                    <div class="p-5">
                        <p class="text-sm text-orange-500 font-medium mb-1">${new Date(event.date).toLocaleDateString()}</p>
                        <h3 class="font-bold text-lg mb-2">${currentLang === 'en' ? event.title_en : event.title_fr}</h3>
                        <p class="text-gray-500 text-sm mb-4"><i class="fas fa-map-marker-alt mr-2"></i>${event.city}</p>
                        <div class="flex items-center justify-between">
                            <div>
                                <p class="text-xs text-gray-400">From</p>
                                <p class="font-bold text-lg">${formatPrice(event.price, event.currency)}</p>
                            </div>
                            <button class="gradient-bg text-white px-4 py-2 rounded-lg font-medium text-sm">Get Tickets</button>
                        </div>
                    </div>
                </div>
            `).join('');
        }

        function openMobileMenu() {
            document.getElementById('mobileMenu').classList.add('open');
            document.getElementById('mobileMenuOverlay').classList.remove('hidden');
            document.body.style.overflow = 'hidden';
        }

        function closeMobileMenu() {
            document.getElementById('mobileMenu').classList.remove('open');
            document.getElementById('mobileMenuOverlay').classList.add('hidden');
            document.body.style.overflow = '';
        }

        function showToast(message, type = 'success') {
            const container = document.getElementById('toastContainer');
            const toast = document.createElement('div');
            const colors = { success: 'bg-green-500', error: 'bg-red-500', info: 'bg-blue-500', warning: 'bg-yellow-500' };
            const icons = { success: 'fa-check-circle', error: 'fa-times-circle', info: 'fa-info-circle', warning: 'fa-exclamation-circle' };
            toast.className = `toast ${colors[type]} text-white px-6 py-4 rounded-xl shadow-lg flex items-center space-x-3 min-w-[300px]`;
            toast.innerHTML = `<i class="fas ${icons[type]}"></i><span>${message}</span>`;
            container.appendChild(toast);
            setTimeout(() => toast.classList.add('show'), 10);
            setTimeout(() => {
                toast.classList.remove('show');
                setTimeout(() => toast.remove(), 300);
            }, 3000);
        }

        function showInviteMemberModal() {
            const modal = document.getElementById('inviteMemberModal');
            modal.classList.remove('hidden');
            modal.classList.add('flex');
            setTimeout(() => modal.classList.add('active'), 10);
        }

        function closeInviteMemberModal() {
            const modal = document.getElementById('inviteMemberModal');
            modal.classList.remove('active');
            setTimeout(() => {
                modal.classList.add('hidden');
                modal.classList.remove('flex');
            }, 300);
        }

        function sendInvite(e) {
            e.preventDefault();
            const email = document.getElementById('inviteEmail').value;
            const role = document.getElementById('inviteRole').value;
            
            const newMember = {
                id: teamMembers.length + 1,
                name: email.split('@')[0],
                email: email,
                role: role,
                initials: email.substring(0, 2).toUpperCase()
            };
            
            teamMembers.push(newMember);
            renderTeamMembers();
            closeInviteMemberModal();
            showToast(`Invitation sent to ${email}!`, 'success');
            document.getElementById('inviteEmail').value = '';
        }

        function renderTeamMembers() {
            const container = document.getElementById('teamMembersList');
            if (!container) return;
            
            const roleColors = {
                owner: 'bg-purple-100 text-purple-600',
                manager: 'bg-blue-100 text-blue-600',
                cashier: 'bg-green-100 text-green-600',
                scanner: 'bg-gray-100 text-gray-600'
            };
            
            container.innerHTML = teamMembers.map(member => `
                <div class="flex items-center justify-between p-4 bg-gray-50 rounded-xl">
                    <div class="flex items-center">
                        <div class="w-12 h-12 gradient-bg rounded-full flex items-center justify-center text-white font-bold mr-4">${member.initials}</div>
                        <div>
                            <p class="font-bold">${member.name}</p>
                            <p class="text-sm text-gray-500">${member.email}</p>
                        </div>
                    </div>
                    <div class="flex items-center space-x-3">
                        <span class="${roleColors[member.role]} px-3 py-1 rounded-full text-sm font-medium capitalize">${member.role}</span>
                        ${member.role !== 'owner' ? `<button onclick="removeMember(${member.id})" class="text-red-500 hover:text-red-700"><i class="fas fa-trash"></i></button>` : ''}
                    </div>
                </div>
            `).join('');
        }

        function removeMember(id) {
            if (confirm('Are you sure you want to remove this team member?')) {
                teamMembers = teamMembers.filter(m => m.id !== id);
                renderTeamMembers();
                showToast('Team member removed', 'info');
            }
        }

        function updateEventCurrency() {
            const select = document.getElementById('eventCountrySelect');
            if (!select) return;
            const country = countries[select.value];
            if (country) {
                const currency1 = document.getElementById('ticketCurrency1');
                if (currency1) currency1.textContent = country.currency;
            }
        }

        function updateSMSCount() {
            const textarea = document.getElementById('smsBody');
            if (!textarea) return;
            const text = textarea.value;
            const length = text.length;
            const charCount = document.getElementById('smsCharCount');
            const segments = document.getElementById('smsSegments');
            const preview = document.getElementById('smsPreview');
            
            if (charCount) charCount.textContent = `${length}/160 characters`;
            if (segments) segments.textContent = `${Math.ceil(length / 160) || 1} SMS segment${length > 160 ? 's' : ''}`;
            if (preview) preview.textContent = text || 'Your SMS message will appear here...';
        }

        function copyReferralLink() {
            const link = 'https://passafrik.com/ref/AFE2025';
            navigator.clipboard.writeText(link).then(() => {
                showToast('Referral link copied to clipboard!', 'success');
            });
        }

        // Initialize
        document.addEventListener('DOMContentLoaded', function() {
            renderCountrySelectors();
            renderEvents();
            renderTeamMembers();
            setLanguage('en');
        });
    </script>
</body>
</html>
