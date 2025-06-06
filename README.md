## User
  Tài liệu này tóm tắt hệ thống prompt chi tiết đã được xây dựng trước đó, nhằm hướng dẫn AI READY trong quá trình phát triển ứng dụng web vẽ sơ đồ mạch logic. Mục tiêu là tạo ra một ứng dụng mạnh mẽ, dễ bảo trì, với kiến trúc thanh lịch. Việc AI READY tuân thủ nghiêm ngặt các chỉ dẫn về kiến trúc, quy tắc đặt tên và các đặc tả chi tiết dưới đây là yếu tố then chốt cho sự thành công của dự án.
  
  Phần 1: Nguyên Tắc Nền Tảng và Hướng Dẫn Phát Triển
  1.1. Tổng Quan Dự Án
  
  AI READY sẽ xây dựng một ứng dụng cho phép người dùng:
  
  Thiết kế sơ đồ mạch logic bằng cách kéo thả các cổng (AND, OR, NOT,...) và các phần tử khác lên một bề mặt vẽ SVG.
  Kết nối các cổng logic bằng dây dẫn.
  Lưu thiết kế ra file và tải lại thiết kế từ file.
  1.2. Triết Lý Kiến Trúc
  
  Ứng dụng sẽ được xây dựng dựa trên sự kết hợp của các triết lý kiến trúc sau, với mục tiêu đạt được sự rõ ràng và trừu tượng tối đa thông qua quy tắc "MỘT TỪ ĐƠN TIẾNG ANH" cho mọi định danh:
  
  Contextual Unigram Architecture: Ý nghĩa của mỗi định danh đơn từ được làm rõ bởi ngữ cảnh của nó (vị trí trong cấu trúc thư mục, module hoặc class chứa nó).
  Role-Centric Unigram Architecture (RCUA): Các module hoặc class được đặt tên theo vai trò hoặc trách nhiệm chính mà chúng đảm nhiệm trong hệ thống.
  Type-Centric Unigram Architecture (TyCUA): Các thực thể dữ liệu hoặc đối tượng trung tâm trong miền vấn đề (như Gate, Wire, Port) được định nghĩa là các Type (kiểu) riêng biệt.
  Trait-Centric Unigram Architecture (TCUA): Các hành vi hoặc năng lực chung (như khả năng kéo thả, khả năng kết nối) được cung cấp bởi các module riêng biệt (trong thư mục app/behavior/) và được tích hợp vào các Element thông qua phương pháp composition.

## Agent
  
  1. Cấu Trúc Tổng Thể
  Trang web được chia thành 3 phần chính:
      * 		Header chứa logo và các công cụ chính
      * 		Sidebar bên trái chứa thư viện các cổng logic
      * 		Khu vực vẽ chính ở giữa màn hình
  2. Header
      * 		Thanh header có nền màu tối, chiều cao 60px
      * 		Bên trái là logo Logic Circuit Designer
      * 		Bên phải là các nút chức năng: + Nút Lưu để lưu thiết kế
      * 		Nút Tải lên để mở file thiết kế có sẵn + Nút Xóa để xóa toàn bộ thiết kế hiện tại
  3. Sidebar
      * 		Chiều rộng 250px, nền màu sáng
      * 		Chứa danh sách các cổng logic có thể kéo thả:
          * 		Cổng AND + Cổng OR+ Cổng NOT
          * 		Cổng NAND
          * 		Cổng NOR
          * 		Cổng XOR
      * 		Mỗi cổng được hiển thị dưới dạng icon và tên
  4. Khu Vực Vẽ
      * 		Chiếm phần còn lại của màn hình
      * 		Nền lưới để dễ định vị
      * 		Cho phép: + Kéo thả các cổng logic từ sidebar
          * 		Vẽ đường nối giữa các cổng
          * 		Di chuyển/xoay các cổng
          * 		Xóa các phần tử
  5. Tương Tác
      * 		Hỗ trợ kéo thả mượt mà
      * 		Tự động căn chỉnh với lưới
      * 		Hiển thị điểm kết nối khi hover
      * 		Có thể multi-select để thao tác nhiều phần tử
      * 		Undo/Redo cho các thao tác
  
  
  ```html
  <!DOCTYPE html>
  <html lang="vi">
  <head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Logic Circuit Designer</title>
  <script src="https://cdn.tailwindcss.com/3.4.16"></script>
  <script>
  // Circuit Model Classes
  class Gate {
  constructor(type, id, x, y) {
  this.type = type;
  this.id = id;
  this.x = x;
  this.y = y;
  this.rotation = 0;
  this.inputs = [];
  this.outputs = [];
  this.state = false;
  }
  evaluate() {
  switch(this.type) {
  case 'AND':
  this.state = this.inputs.every(input => input.state);
  break;
  case 'OR':
  this.state = this.inputs.some(input => input.state);
  break;
  case 'NOT':
  this.state = !this.inputs[0]?.state;
  break;
  case 'NAND':
  this.state = !this.inputs.every(input => input.state);
  break;
  case 'NOR':
  this.state = !this.inputs.some(input => input.state);
  break;
  case 'XOR':
  this.state = this.inputs.reduce((acc, input) => acc ^ input.state, false);
  break;
  case 'INPUT':
  // State is managed externally
  break;
  case 'OUTPUT':
  this.state = this.inputs[0]?.state || false;
  break;
  }
  return this.state;
  }
  }
  class Wire {
  constructor(id, startGate, startPoint, endGate, endPoint) {
  this.id = id;
  this.startGate = startGate;
  this.startPoint = startPoint;
  this.endGate = endGate;
  this.endPoint = endPoint;
  this.state = false;
  }
  updateState() {
  this.state = this.startGate.state;
  return this.state;
  }
  }
  class Circuit {
  constructor() {
  this.gates = new Map();
  this.wires = new Map();
  this.nextGateId = 1;
  this.nextWireId = 1;
  }
  addGate(type, x, y) {
  const id = `gate-${this.nextGateId++}`;
  const gate = new Gate(type, id, x, y);
  this.gates.set(id, gate);
  return gate;
  }
  addWire(startGate, startPoint, endGate, endPoint) {
  const id = `wire-${this.nextWireId++}`;
  const wire = new Wire(id, startGate, startPoint, endGate, endPoint);
  this.wires.set(id, wire);
  // Connect gates
  endGate.inputs.push(startGate);
  startGate.outputs.push(endGate);
  return wire;
  }
  removeGate(gateId) {
  const gate = this.gates.get(gateId);
  if (!gate) return;
  // Remove connected wires
  for (const [wireId, wire] of this.wires) {
  if (wire.startGate === gate || wire.endGate === gate) {
  this.removeWire(wireId);
  }
  }
  // Remove gate
  this.gates.delete(gateId);
  }
  removeWire(wireId) {
  const wire = this.wires.get(wireId);
  if (!wire) return;
  // Disconnect gates
  const startIdx = wire.endGate.inputs.indexOf(wire.startGate);
  if (startIdx > -1) wire.endGate.inputs.splice(startIdx, 1);
  const endIdx = wire.startGate.outputs.indexOf(wire.endGate);
  if (endIdx > -1) wire.startGate.outputs.splice(endIdx, 1);
  this.wires.delete(wireId);
  }
  evaluate() {
  // Topological sort for evaluation order
  const visited = new Set();
  const evalOrder = [];
  const visit = (gate) => {
  if (visited.has(gate)) return;
  visited.add(gate);
  gate.inputs.forEach(input => visit(input));
  evalOrder.push(gate);
  };
  this.gates.forEach(gate => {
  if (gate.type === 'OUTPUT') visit(gate);
  });
  // Evaluate gates in order
  evalOrder.forEach(gate => gate.evaluate());
  // Update wire states
  this.wires.forEach(wire => wire.updateState());
  }
  setInputState(gateId, state) {
  const gate = this.gates.get(gateId);
  if (gate && gate.type === 'INPUT') {
  gate.state = state;
  this.evaluate();
  }
  }
  getOutputState(gateId) {
  const gate = this.gates.get(gateId);
  return gate ? gate.state : false;
  }
  }
  // Initialize circuit
  const circuit = new Circuit();
  </script>
  <script>tailwind.config={theme:{extend:{colors:{primary:'#3b82f6',secondary:'#10b981'},borderRadius:{'none':'0px','sm':'4px',DEFAULT:'8px','md':'12px','lg':'16px','xl':'20px','2xl':'24px','3xl':'32px','full':'9999px','button':'8px'}}}}</script>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Pacifico&display=swap" rel="stylesheet">
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/remixicon/4.6.0/remixicon.min.css">
  <style>
  :where([class^="ri-"])::before { content: "\f3c2"; }
  .grid-bg {
  background-size: 20px 20px;
  background-image:
  linear-gradient(to right, rgba(0, 0, 0, 0.05) 1px, transparent 1px),
  linear-gradient(to bottom, rgba(0, 0, 0, 0.05) 1px, transparent 1px);
  }
  .gate {
  cursor: move;
  user-select: none;
  }
  .connection-point {
  fill: #fff;
  stroke: #3b82f6;
  stroke-width: 2;
  cursor: pointer;
  }
  .connection-point:hover {
  fill: #3b82f6;
  }
  .wire {
  stroke: #3b82f6;
  stroke-width: 2;
  fill: none;
  }
  input[type="range"]::-webkit-slider-thumb {
  appearance: none;
  width: 16px;
  height: 16px;
  background-color: #3b82f6;
  border-radius: 50%;
  cursor: pointer;
  }
  input[type="range"] {
  appearance: none;
  height: 4px;
  background: #e5e7eb;
  border-radius: 4px;
  outline: none;
  }
  </style>
  </head>
  <body class="min-h-screen flex flex-col bg-gray-50">
  <!-- Header -->
  <header class="bg-gray-800 text-white h-[60px] flex items-center justify-between px-4 shadow-md">
  <div class="text-xl font-['Pacifico'] text-white">Logic Circuit Designer</div>
  <div class="flex items-center space-x-3">
  <button id="save-btn" class="flex items-center space-x-1 bg-primary hover:bg-blue-600 text-white px-3 py-2 !rounded-button whitespace-nowrap">
  <div class="w-5 h-5 flex items-center justify-center">
  <i class="ri-save-line"></i>
  </div>
  <span>Lưu</span>
  </button>
  <button id="load-btn" class="flex items-center space-x-1 bg-green-600 hover:bg-green-700 text-white px-3 py-2 !rounded-button whitespace-nowrap">
  <div class="w-5 h-5 flex items-center justify-center">
  <i class="ri-upload-line"></i>
  </div>
  <span>Tải lên</span>
  </button>
  <button id="clear-btn" class="flex items-center space-x-1 bg-red-600 hover:bg-red-700 text-white px-3 py-2 !rounded-button whitespace-nowrap">
  <div class="w-5 h-5 flex items-center justify-center">
  <i class="ri-delete-bin-line"></i>
  </div>
  <span>Xóa</span>
  </button>
  </div>
  </header>
  <!-- Main Content -->
  <div class="flex flex-1 overflow-hidden">
  <!-- Sidebar -->
  <aside class="w-[250px] bg-white border-r border-gray-200 p-4 overflow-y-auto">
  <h2 class="font-semibold text-lg mb-4">Thư viện cổng logic</h2>
  <div class="space-y-3">
  <div class="gate-item gate p-3 bg-gray-50 hover:bg-gray-100 rounded flex items-center space-x-3 cursor-move" data-gate-type="AND">
  <svg width="40" height="40" viewBox="0 0 40 40" class="flex-shrink-0">
  <path d="M10,5 L10,35 L20,35 C27,35 30,20 30,20 C30,20 27,5 20,5 L10,5 Z" fill="white" stroke="#3b82f6" stroke-width="2"></path>
  <circle cx="5" cy="12" r="2" fill="#3b82f6"></circle>
  <circle cx="5" cy="28" r="2" fill="#3b82f6"></circle>
  <circle cx="35" cy="20" r="2" fill="#3b82f6"></circle>
  </svg>
  <span>Cổng AND</span>
  </div>
  <div class="gate-item gate p-3 bg-gray-50 hover:bg-gray-100 rounded flex items-center space-x-3 cursor-move" data-gate-type="OR">
  <svg width="40" height="40" viewBox="0 0 40 40" class="flex-shrink-0">
  <path d="M5,5 Q20,5 30,20 Q20,35 5,35 Z" fill="white" stroke="#3b82f6" stroke-width="2"></path>
  <circle cx="5" cy="12" r="2" fill="#3b82f6"></circle>
  <circle cx="5" cy="28" r="2" fill="#3b82f6"></circle>
  <circle cx="35" cy="20" r="2" fill="#3b82f6"></circle>
  </svg>
  <span>Cổng OR</span>
  </div>
  <div class="gate-item gate p-3 bg-gray-50 hover:bg-gray-100 rounded flex items-center space-x-3 cursor-move" data-gate-type="NOT">
  <svg width="40" height="40" viewBox="0 0 40 40" class="flex-shrink-0">
  <path d="M10,5 L30,20 L10,35 Z" fill="white" stroke="#3b82f6" stroke-width="2"></path>
  <circle cx="35" cy="20" r="3" fill="white" stroke="#3b82f6" stroke-width="2"></circle>
  <circle cx="5" cy="20" r="2" fill="#3b82f6"></circle>
  </svg>
  <span>Cổng NOT</span>
  </div>
  <div class="gate-item gate p-3 bg-gray-50 hover:bg-gray-100 rounded flex items-center space-x-3 cursor-move" data-gate-type="NAND">
  <svg width="40" height="40" viewBox="0 0 40 40" class="flex-shrink-0">
  <path d="M10,5 L10,35 L20,35 C27,35 30,20 30,20 C30,20 27,5 20,5 L10,5 Z" fill="white" stroke="#3b82f6" stroke-width="2"></path>
  <circle cx="35" cy="20" r="3" fill="white" stroke="#3b82f6" stroke-width="2"></circle>
  <circle cx="5" cy="12" r="2" fill="#3b82f6"></circle>
  <circle cx="5" cy="28" r="2" fill="#3b82f6"></circle>
  </svg>
  <span>Cổng NAND</span>
  </div>
  <div class="gate-item gate p-3 bg-gray-50 hover:bg-gray-100 rounded flex items-center space-x-3 cursor-move" data-gate-type="NOR">
  <svg width="40" height="40" viewBox="0 0 40 40" class="flex-shrink-0">
  <path d="M5,5 Q20,5 30,20 Q20,35 5,35 Z" fill="white" stroke="#3b82f6" stroke-width="2"></path>
  <circle cx="35" cy="20" r="3" fill="white" stroke="#3b82f6" stroke-width="2"></circle>
  <circle cx="5" cy="12" r="2" fill="#3b82f6"></circle>
  <circle cx="5" cy="28" r="2" fill="#3b82f6"></circle>
  </svg>
  <span>Cổng NOR</span>
  </div>
  <div class="gate-item gate p-3 bg-gray-50 hover:bg-gray-100 rounded flex items-center space-x-3 cursor-move" data-gate-type="XOR">
  <svg width="40" height="40" viewBox="0 0 40 40" class="flex-shrink-0">
  <path d="M10,5 Q25,5 35,20 Q25,35 10,35" fill="none" stroke="#3b82f6" stroke-width="2"></path>
  <path d="M5,5 Q20,5 30,20 Q20,35 5,35 Z" fill="white" stroke="#3b82f6" stroke-width="2"></path>
  <circle cx="5" cy="12" r="2" fill="#3b82f6"></circle>
  <circle cx="5" cy="28" r="2" fill="#3b82f6"></circle>
  <circle cx="35" cy="20" r="2" fill="#3b82f6"></circle>
  </svg>
  <span>Cổng XOR</span>
  </div>
  <div class="gate-item gate p-3 bg-gray-50 hover:bg-gray-100 rounded flex items-center space-x-3 cursor-move" data-gate-type="INPUT">
  <svg width="40" height="40" viewBox="0 0 40 40" class="flex-shrink-0">
  <rect x="10" y="15" width="20" height="10" fill="white" stroke="#3b82f6" stroke-width="2" rx="2"></rect>
  <circle cx="35" cy="20" r="2" fill="#3b82f6"></circle>
  <text x="20" y="23" font-size="8" text-anchor="middle" fill="#3b82f6">INPUT</text>
  </svg>
  <span>Đầu vào</span>
  </div>
  <div class="gate-item gate p-3 bg-gray-50 hover:bg-gray-100 rounded flex items-center space-x-3 cursor-move" data-gate-type="OUTPUT">
  <svg width="40" height="40" viewBox="0 0 40 40" class="flex-shrink-0">
  <rect x="10" y="15" width="20" height="10" fill="white" stroke="#3b82f6" stroke-width="2" rx="2"></rect>
  <circle cx="5" cy="20" r="2" fill="#3b82f6"></circle>
  <text x="20" y="23" font-size="8" text-anchor="middle" fill="#3b82f6">OUTPUT</text>
  </svg>
  <span>Đầu ra</span>
  </div>
  </div>
  <div class="mt-8 border-t pt-4">
  <h2 class="font-semibold text-lg mb-4">Công cụ</h2>
  <div class="space-y-3">
  <div class="flex items-center justify-between">
  <label class="font-medium">Kích thước lưới:</label>
  <input type="range" id="grid-size" min="10" max="50" value="20" class="w-24">
  </div>
  <div class="flex items-center space-x-2">
  <div class="w-5 h-5 flex items-center justify-center">
  <i class="ri-cursor-line"></i>
  </div>
  <button id="select-tool" class="bg-primary text-white px-3 py-2 !rounded-button whitespace-nowrap">Chọn</button>
  </div>
  <div class="flex items-center space-x-2">
  <div class="w-5 h-5 flex items-center justify-center">
  <i class="ri-link-m"></i>
  </div>
  <button id="connect-tool" class="bg-gray-200 hover:bg-gray-300 px-3 py-2 !rounded-button whitespace-nowrap">Kết nối</button>
  </div>
  <div class="flex items-center space-x-2">
  <div class="w-5 h-5 flex items-center justify-center">
  <i class="ri-delete-bin-line"></i>
  </div>
  <button id="delete-tool" class="bg-gray-200 hover:bg-gray-300 px-3 py-2 !rounded-button whitespace-nowrap">Xóa</button>
  </div>
  </div>
  </div>
  </aside>
  <!-- Drawing Area -->
  <main class="flex-1 overflow-auto relative">
  <div id="canvas-container" class="w-full h-full grid-bg">
  <svg id="canvas" width="100%" height="100%" class="absolute top-0 left-0">
  <g id="wires"></g>
  <g id="gates"></g>
  <g id="selection-box" class="hidden">
  <rect x="0" y="0" width="0" height="0" fill="rgba(59, 130, 246, 0.1)" stroke="#3b82f6" stroke-width="1" stroke-dasharray="5,5"></rect>
  </g>
  </svg>
  </div>
  </main>
  </div>
  <!-- Context Menu -->
  <div id="context-menu" class="hidden absolute bg-white shadow-lg rounded-md overflow-hidden z-50 w-48 border border-gray-200">
  <div class="py-1">
  <button class="w-full text-left px-4 py-2 hover:bg-gray-100 flex items-center" id="rotate-element">
  <div class="w-5 h-5 flex items-center justify-center mr-2">
  <i class="ri-refresh-line"></i>
  </div>
  <span>Xoay</span>
  </button>
  <button class="w-full text-left px-4 py-2 hover:bg-gray-100 flex items-center" id="duplicate-element">
  <div class="w-5 h-5 flex items-center justify-center mr-2">
  <i class="ri-file-copy-line"></i>
  </div>
  <span>Nhân bản</span>
  </button>
  <button class="w-full text-left px-4 py-2 hover:bg-gray-100 flex items-center text-red-600" id="delete-element">
  <div class="w-5 h-5 flex items-center justify-center mr-2">
  <i class="ri-delete-bin-line"></i>
  </div>
  <span>Xóa</span>
  </button>
  </div>
  </div>
  <!-- Modal for Save/Load -->
  <div id="modal" class="hidden fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
  <div class="bg-white rounded-lg shadow-xl p-6 w-96">
  <div class="flex justify-between items-center mb-4">
  <h3 class="text-lg font-semibold" id="modal-title">Lưu thiết kế</h3>
  <button id="close-modal" class="text-gray-500 hover:text-gray-700">
  <div class="w-6 h-6 flex items-center justify-center">
  <i class="ri-close-line"></i>
  </div>
  </button>
  </div>
  <div id="modal-content">
  <div id="save-form">
  <div class="mb-4">
  <label for="design-name" class="block mb-2 text-sm font-medium">Tên thiết kế</label>
  <input type="text" id="design-name" class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-primary" placeholder="Nhập tên thiết kế">
  </div>
  <div class="flex justify-end">
  <button id="confirm-save" class="bg-primary hover:bg-blue-600 text-white px-4 py-2 !rounded-button whitespace-nowrap">Lưu</button>
  </div>
  </div>
  <div id="load-form" class="hidden">
  <div class="mb-4">
  <label class="block mb-2 text-sm font-medium">Chọn thiết kế</label>
  <select id="saved-designs" class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-primary pr-8">
  <option value="">-- Chọn thiết kế --</option>
  <option value="design1">Mạch đếm nhị phân</option>
  <option value="design2">Mạch cộng 1 bit</option>
  <option value="design3">Mạch giải mã 2:4</option>
  </select>
  </div>
  <div class="flex justify-end">
  <button id="confirm-load" class="bg-primary hover:bg-blue-600 text-white px-4 py-2 !rounded-button whitespace-nowrap">Tải lên</button>
  </div>
  </div>
  </div>
  </div>
  </div>
  <script id="canvas-interaction">
  document.addEventListener('DOMContentLoaded', function() {
  const canvas = document.getElementById('canvas');
  const canvasContainer = document.getElementById('canvas-container');
  const gatesGroup = document.getElementById('gates');
  const wiresGroup = document.getElementById('wires');
  const selectionBox = document.getElementById('selection-box');
  const selectionRect = selectionBox.querySelector('rect');
  let currentTool = 'select';
  let isDragging = false;
  let isConnecting = false;
  let isSelecting = false;
  let draggedElement = null;
  let selectedElements = [];
  let startPoint = { x: 0, y: 0 };
  let connectionStartPoint = null;
  let connectionEndPoint = null;
  let gridSize = 20;
  let nextGateId = 1;
  let nextWireId = 1;
  let scale = 1;
  let panOffset = { x: 0, y: 0 };
  let isPanning = false;
  let lastMousePos = { x: 0, y: 0 };
  let undoStack = [];
  let redoStack = [];
  // Set tool buttons
  document.getElementById('select-tool').addEventListener('click', () => setTool('select'));
  document.getElementById('connect-tool').addEventListener('click', () => setTool('connect'));
  document.getElementById('delete-tool').addEventListener('click', () => setTool('delete'));
  function setTool(tool) {
  currentTool = tool;
  document.getElementById('select-tool').classList.remove('bg-primary', 'text-white');
  document.getElementById('select-tool').classList.add('bg-gray-200', 'hover:bg-gray-300');
  document.getElementById('connect-tool').classList.remove('bg-primary', 'text-white');
  document.getElementById('connect-tool').classList.add('bg-gray-200', 'hover:bg-gray-300');
  document.getElementById('delete-tool').classList.remove('bg-primary', 'text-white');
  document.getElementById('delete-tool').classList.add('bg-gray-200', 'hover:bg-gray-300');
  if (tool === 'select') {
  document.getElementById('select-tool').classList.remove('bg-gray-200', 'hover:bg-gray-300');
  document.getElementById('select-tool').classList.add('bg-primary', 'text-white');
  } else if (tool === 'connect') {
  document.getElementById('connect-tool').classList.remove('bg-gray-200', 'hover:bg-gray-300');
  document.getElementById('connect-tool').classList.add('bg-primary', 'text-white');
  } else if (tool === 'delete') {
  document.getElementById('delete-tool').classList.remove('bg-gray-200', 'hover:bg-gray-300');
  document.getElementById('delete-tool').classList.add('bg-primary', 'text-white');
  }
  // Clear any current selection
  clearSelection();
  }
  // Grid size control
  document.getElementById('grid-size').addEventListener('input', function(e) {
  gridSize = parseInt(e.target.value);
  updateGridBackground();
  });
  function updateGridBackground() {
  canvasContainer.style.backgroundSize = `${gridSize}px ${gridSize}px`;
  }
  // Make gates draggable from sidebar
  const gateItems = document.querySelectorAll('.gate-item');
  gateItems.forEach(item => {
  item.addEventListener('mousedown', function(e) {
  const gateType = this.getAttribute('data-gate-type');
  const svgContent = this.querySelector('svg').outerHTML;
  // Create a clone for dragging
  const clone = document.createElement('div');
  clone.innerHTML = svgContent;
  clone.style.position = 'absolute';
  clone.style.left = e.clientX + 'px';
  clone.style.top = e.clientY + 'px';
  clone.style.pointerEvents = 'none';
  clone.style.zIndex = '1000';
  clone.setAttribute('data-gate-type', gateType);
  document.body.appendChild(clone);
  // Set up drag
  draggedElement = {
  element: clone,
  type: 'new-gate',
  gateType: gateType,
  offsetX: 20,
  offsetY: 20
  };
  isDragging = true;
  document.addEventListener('mousemove', onMouseMove);
  document.addEventListener('mouseup', onMouseUp);
  e.preventDefault();
  });
  });
  // Canvas mouse events
  canvas.addEventListener('mousedown', function(e) {
  if (e.button === 1) { // Middle mouse button
  e.preventDefault();
  isPanning = true;
  lastMousePos = { x: e.clientX, y: e.clientY };
  return;
  }
  if (e.button !== 0) return; // Only left mouse button
  const rect = canvas.getBoundingClientRect();
  const x = e.clientX - rect.left;
  const y = e.clientY - rect.top;
  startPoint = { x, y };
  // Check for connection point first, regardless of current tool
  const connectionPoint = findConnectionPointAtPosition(x, y);
  if (connectionPoint) {
  isConnecting = true;
  connectionStartPoint = {
  element: connectionPoint.gate,
  point: connectionPoint.point,
  x: parseFloat(connectionPoint.gate.getAttribute('data-x')) + connectionPoint.point.x,
  y: parseFloat(connectionPoint.gate.getAttribute('data-y')) + connectionPoint.point.y,
  type: connectionPoint.point.type
  };
  // Switch to connect tool automatically
  setTool('connect');
  return;
  }
  if (currentTool === 'select') {
  // Check if clicked on a gate
  const clickedGate = findGateAtPosition(x, y);
  if (clickedGate) {
  if (!e.ctrlKey && !selectedElements.includes(clickedGate)) {
  clearSelection();
  }
  if (!selectedElements.includes(clickedGate)) {
  selectElement(clickedGate);
  }
  draggedElement = {
  element: clickedGate,
  type: 'existing-gate',
  offsetX: x - parseFloat(clickedGate.getAttribute('data-x')),
  offsetY: y - parseFloat(clickedGate.getAttribute('data-y'))
  };
  isDragging = true;
  } else {
  // Check if clicked on a connection point
  const connectionPoint = findConnectionPointAtPosition(x, y);
  if (connectionPoint && currentTool === 'connect') {
  isConnecting = true;
  connectionStartPoint = {
  element: connectionPoint.gate,
  point: connectionPoint.point,
  x: parseFloat(connectionPoint.gate.getAttribute('data-x')) + connectionPoint.point.x,
  y: parseFloat(connectionPoint.gate.getAttribute('data-y')) + connectionPoint.point.y
  };
  } else {
  // Start selection box
  isSelecting = true;
  selectionBox.classList.remove('hidden');
  selectionRect.setAttribute('x', x);
  selectionRect.setAttribute('y', y);
  selectionRect.setAttribute('width', 0);
  selectionRect.setAttribute('height', 0);
  if (!e.ctrlKey) {
  clearSelection();
  }
  }
  }
  } else if (currentTool === 'connect') {
  const connectionPoint = findConnectionPointAtPosition(x, y);
  if (connectionPoint) {
  isConnecting = true;
  connectionStartPoint = {
  element: connectionPoint.gate,
  point: connectionPoint.point,
  x: parseFloat(connectionPoint.gate.getAttribute('data-x')) + connectionPoint.point.x,
  y: parseFloat(connectionPoint.gate.getAttribute('data-y')) + connectionPoint.point.y
  };
  }
  } else if (currentTool === 'delete') {
  const clickedGate = findGateAtPosition(x, y);
  if (clickedGate) {
  removeGate(clickedGate);
  }
  const clickedWire = findWireAtPosition(x, y);
  if (clickedWire) {
  removeWire(clickedWire);
  }
  }
  });
  canvas.addEventListener('mousemove', onMouseMove);
  canvas.addEventListener('mouseup', onMouseUp);
  canvas.addEventListener('wheel', function(e) {
  e.preventDefault();
  const delta = e.deltaY > 0 ? 0.9 : 1.1;
  const oldScale = scale;
  scale *= delta;
  scale = Math.min(Math.max(0.5, scale), 2); // Limit zoom between 0.5x and 2x
  
  const rect = canvas.getBoundingClientRect();
  const mouseX = e.clientX - rect.left;
  const mouseY = e.clientY - rect.top;
  
  panOffset.x += mouseX * (1 - scale/oldScale);
  panOffset.y += mouseY * (1 - scale/oldScale);
  
  gatesGroup.setAttribute('transform', `translate(${panOffset.x},${panOffset.y}) scale(${scale})`);
  wiresGroup.setAttribute('transform', `translate(${panOffset.x},${panOffset.y}) scale(${scale})`);
  });
  
  function saveState() {
  const state = {
  gates: gatesGroup.innerHTML,
  wires: wiresGroup.innerHTML
  };
  undoStack.push(state);
  redoStack = [];
  updateUndoRedoButtons();
  }
  
  function undo() {
  if (undoStack.length === 0) return;
  const currentState = {
  gates: gatesGroup.innerHTML,
  wires: wiresGroup.innerHTML
  };
  redoStack.push(currentState);
  const previousState = undoStack.pop();
  gatesGroup.innerHTML = previousState.gates;
  wiresGroup.innerHTML = previousState.wires;
  updateUndoRedoButtons();
  }
  
  function redo() {
  if (redoStack.length === 0) return;
  const currentState = {
  gates: gatesGroup.innerHTML,
  wires: wiresGroup.innerHTML
  };
  undoStack.push(currentState);
  const nextState = redoStack.pop();
  gatesGroup.innerHTML = nextState.gates;
  wiresGroup.innerHTML = nextState.wires;
  updateUndoRedoButtons();
  }
  
  function updateUndoRedoButtons() {
  document.getElementById('undo-btn').disabled = undoStack.length === 0;
  document.getElementById('redo-btn').disabled = redoStack.length === 0;
  }
  
  document.getElementById('undo-btn').addEventListener('click', undo);
  document.getElementById('redo-btn').addEventListener('click', redo);
  
  // Add keyboard shortcuts
  document.addEventListener('keydown', function(e) {
  if (e.ctrlKey || e.metaKey) {
  if (e.key === 'z') {
  e.preventDefault();
  if (e.shiftKey) {
  redo();
  } else {
  undo();
  }
  }
  }
  });
  function onMouseMove(e) {
  if (!isDragging && !isConnecting && !isSelecting && !isPanning) return;
  const rect = canvas.getBoundingClientRect();
  const x = (e.clientX - rect.left - panOffset.x) / scale;
  const y = (e.clientY - rect.top - panOffset.y) / scale;
  
  if (isPanning) {
  const dx = e.clientX - lastMousePos.x;
  const dy = e.clientY - lastMousePos.y;
  panOffset.x += dx;
  panOffset.y += dy;
  lastMousePos = { x: e.clientX, y: e.clientY };
  gatesGroup.setAttribute('transform', `translate(${panOffset.x},${panOffset.y}) scale(${scale})`);
  wiresGroup.setAttribute('transform', `translate(${panOffset.x},${panOffset.y}) scale(${scale})`);
  return;
  }
  if (isDragging && draggedElement) {
  if (draggedElement.type === 'new-gate') {
  draggedElement.element.style.left = (e.clientX - draggedElement.offsetX) + 'px';
  draggedElement.element.style.top = (e.clientY - draggedElement.offsetY) + 'px';
  } else if (draggedElement.type === 'existing-gate') {
  // Snap to grid
  const snappedX = Math.round((x - draggedElement.offsetX) / gridSize) * gridSize;
  const snappedY = Math.round((y - draggedElement.offsetY) / gridSize) * gridSize;
  draggedElement.element.setAttribute('data-x', snappedX);
  draggedElement.element.setAttribute('data-y', snappedY);
  draggedElement.element.setAttribute('transform', `translate(${snappedX}, ${snappedY})`);
  // Move all selected elements
  if (selectedElements.length > 1) {
  const dx = snappedX - parseFloat(draggedElement.element.getAttribute('data-last-x') || snappedX);
  const dy = snappedY - parseFloat(draggedElement.element.getAttribute('data-last-y') || snappedY);
  if (dx !== 0 || dy !== 0) {
  selectedElements.forEach(element => {
  if (element !== draggedElement.element) {
  const currentX = parseFloat(element.getAttribute('data-x'));
  const currentY = parseFloat(element.getAttribute('data-y'));
  const newX = currentX + dx;
  const newY = currentY + dy;
  element.setAttribute('data-x', newX);
  element.setAttribute('data-y', newY);
  element.setAttribute('transform', `translate(${newX}, ${newY})`);
  }
  });
  }
  draggedElement.element.setAttribute('data-last-x', snappedX);
  draggedElement.element.setAttribute('data-last-y', snappedY);
  }
  // Update connected wires
  updateConnectedWires(draggedElement.element);
  }
  } else if (isConnecting && connectionStartPoint) {
  // Draw temporary connection line
  let tempLine = document.getElementById('temp-connection-line');
  if (!tempLine) {
  tempLine = document.createElementNS('http://www.w3.org/2000/svg', 'path');
  tempLine.setAttribute('id', 'temp-connection-line');
  tempLine.setAttribute('stroke', '#3b82f6');
  tempLine.setAttribute('stroke-width', '2');
  tempLine.setAttribute('fill', 'none');
  tempLine.setAttribute('stroke-dasharray', '5,5');
  wiresGroup.appendChild(tempLine);
  }
  const connectionPoint = findConnectionPointAtPosition(x, y);
  if (connectionPoint) {
  connectionEndPoint = {
  element: connectionPoint.gate,
  point: connectionPoint.point,
  x: parseFloat(connectionPoint.gate.getAttribute('data-x')) + connectionPoint.point.x,
  y: parseFloat(connectionPoint.gate.getAttribute('data-y')) + connectionPoint.point.y
  };
  // Highlight valid connection point
  const circles = document.querySelectorAll('.connection-point');
  circles.forEach(circle => circle.setAttribute('fill', '#fff'));
  const pointElement = connectionPoint.gate.querySelector(`circle[cx="${connectionPoint.point.x}"][cy="${connectionPoint.point.y}"]`);
  if (pointElement) {
  pointElement.setAttribute('fill', '#3b82f6');
  }
  } else {
  connectionEndPoint = { x, y };
  // Remove highlight
  const circles = document.querySelectorAll('.connection-point');
  circles.forEach(circle => circle.setAttribute('fill', '#fff'));
  }
  // Draw path
  const path = generatePath(connectionStartPoint, connectionEndPoint);
  tempLine.setAttribute('d', path);
  } else if (isSelecting) {
  // Update selection box
  const width = x - startPoint.x;
  const height = y - startPoint.y;
  if (width < 0) {
  selectionRect.setAttribute('x', x);
  selectionRect.setAttribute('width', -width);
  } else {
  selectionRect.setAttribute('width', width);
  }
  if (height < 0) {
  selectionRect.setAttribute('y', y);
  selectionRect.setAttribute('height', -height);
  } else {
  selectionRect.setAttribute('height', height);
  }
  // Select gates inside the selection box
  const selX = parseFloat(selectionRect.getAttribute('x'));
  const selY = parseFloat(selectionRect.getAttribute('y'));
  const selWidth = parseFloat(selectionRect.getAttribute('width'));
  const selHeight = parseFloat(selectionRect.getAttribute('height'));
  const gates = document.querySelectorAll('#gates > g');
  gates.forEach(gate => {
  const gateX = parseFloat(gate.getAttribute('data-x'));
  const gateY = parseFloat(gate.getAttribute('data-y'));
  if (gateX >= selX && gateX <= selX + selWidth &&
  gateY >= selY && gateY <= selY + selHeight) {
  if (!selectedElements.includes(gate)) {
  selectElement(gate);
  }
  } else if (!e.ctrlKey && selectedElements.includes(gate)) {
  deselectElement(gate);
  }
  });
  }
  }
  function onMouseUp(e) {
  if (isDragging && draggedElement) {
  if (draggedElement.type === 'new-gate') {
  // Check if dropped on canvas
  const rect = canvas.getBoundingClientRect();
  if (e.clientX >= rect.left && e.clientX <= rect.right &&
  e.clientY >= rect.top && e.clientY <= rect.bottom) {
  // Calculate position on canvas
  const x = e.clientX - rect.left;
  const y = e.clientY - rect.top;
  // Snap to grid
  const snappedX = Math.round(x / gridSize) * gridSize;
  const snappedY = Math.round(y / gridSize) * gridSize;
  // Create gate on canvas
  createGate(draggedElement.gateType, snappedX, snappedY);
  }
  // Remove drag clone
  document.body.removeChild(draggedElement.element);
  }
  draggedElement = null;
  isDragging = false;
  }
  if (isConnecting && connectionStartPoint && connectionEndPoint) {
  // Remove temporary line
  const tempLine = document.getElementById('temp-connection-line');
  if (tempLine) {
  tempLine.remove();
  }
  // Check if connecting to a valid point
  if (connectionEndPoint.element && connectionEndPoint.element !== connectionStartPoint.element) {
  createWire(connectionStartPoint, connectionEndPoint);
  }
  // Remove highlight
  const circles = document.querySelectorAll('.connection-point');
  circles.forEach(circle => circle.setAttribute('fill', '#fff'));
  connectionStartPoint = null;
  connectionEndPoint = null;
  isConnecting = false;
  }
  if (isSelecting) {
  selectionBox.classList.add('hidden');
  isSelecting = false;
  }
  document.removeEventListener('mousemove', onMouseMove);
  document.removeEventListener('mouseup', onMouseUp);
  }
  // Helper functions
  function createGate(gateType, x, y) {
  // Create logical gate in circuit
  const logicalGate = circuit.addGate(gateType, x, y);
  const gateId = logicalGate.id;
  const gate = document.createElementNS('http://www.w3.org/2000/svg', 'g');
  gate.setAttribute('id', gateId);
  gate.setAttribute('data-gate-type', gateType);
  gate.setAttribute('data-x', x);
  gate.setAttribute('data-y', y);
  gate.setAttribute('transform', `translate(${x}, ${y})`);
  gate.classList.add('gate');
  let svgContent = '';
  let connectionPoints = [];
  switch (gateType) {
  case 'AND':
  svgContent = `
  <path d="M10,5 L10,35 L20,35 C27,35 30,20 30,20 C30,20 27,5 20,5 L10,5 Z" fill="white" stroke="#3b82f6" stroke-width="2"></path>
  <circle cx="5" cy="12" r="2" class="connection-point" data-type="input"></circle>
  <circle cx="5" cy="28" r="2" class="connection-point" data-type="input"></circle>
  <circle cx="35" cy="20" r="2" class="connection-point" data-type="output"></circle>
  `;
  connectionPoints = [
  { x: 5, y: 12, type: 'input' },
  { x: 5, y: 28, type: 'input' },
  { x: 35, y: 20, type: 'output' }
  ];
  break;
  case 'OR':
  svgContent = `
  <path d="M5,5 Q20,5 30,20 Q20,35 5,35 Z" fill="white" stroke="#3b82f6" stroke-width="2"></path>
  <circle cx="5" cy="12" r="2" class="connection-point" data-type="input"></circle>
  <circle cx="5" cy="28" r="2" class="connection-point" data-type="input"></circle>
  <circle cx="35" cy="20" r="2" class="connection-point" data-type="output"></circle>
  `;
  connectionPoints = [
  { x: 5, y: 12, type: 'input' },
  { x: 5, y: 28, type: 'input' },
  { x: 35, y: 20, type: 'output' }
  ];
  break;
  case 'NOT':
  svgContent = `
  <path d="M10,5 L30,20 L10,35 Z" fill="white" stroke="#3b82f6" stroke-width="2"></path>
  <circle cx="35" cy="20" r="3" fill="white" stroke="#3b82f6" stroke-width="2"></circle>
  <circle cx="5" cy="20" r="2" class="connection-point" data-type="input"></circle>
  <circle cx="38" cy="20" r="2" class="connection-point" data-type="output"></circle>
  `;
  connectionPoints = [
  { x: 5, y: 20, type: 'input' },
  { x: 38, y: 20, type: 'output' }
  ];
  break;
  case 'NAND':
  svgContent = `
  <path d="M10,5 L10,35 L20,35 C27,35 30,20 30,20 C30,20 27,5 20,5 L10,5 Z" fill="white" stroke="#3b82f6" stroke-width="2"></path>
  <circle cx="35" cy="20" r="3" fill="white" stroke="#3b82f6" stroke-width="2"></circle>
  <circle cx="5" cy="12" r="2" class="connection-point" data-type="input"></circle>
  <circle cx="5" cy="28" r="2" class="connection-point" data-type="input"></circle>
  <circle cx="38" cy="20" r="2" class="connection-point" data-type="output"></circle>
  `;
  connectionPoints = [
  { x: 5, y: 12, type: 'input' },
  { x: 5, y: 28, type: 'input' },
  { x: 38, y: 20, type: 'output' }
  ];
  break;
  case 'NOR':
  svgContent = `
  <path d="M5,5 Q20,5 30,20 Q20,35 5,35 Z" fill="white" stroke="#3b82f6" stroke-width="2"></path>
  <circle cx="35" cy="20" r="3" fill="white" stroke="#3b82f6" stroke-width="2"></circle>
  <circle cx="5" cy="12" r="2" class="connection-point" data-type="input"></circle>
  <circle cx="5" cy="28" r="2" class="connection-point" data-type="input"></circle>
  <circle cx="38" cy="20" r="2" class="connection-point" data-type="output"></circle>
  `;
  connectionPoints = [
  { x: 5, y: 12, type: 'input' },
  { x: 5, y: 28, type: 'input' },
  { x: 38, y: 20, type: 'output' }
  ];
  break;
  case 'XOR':
  svgContent = `
  <path d="M10,5 Q25,5 35,20 Q25,35 10,35" fill="none" stroke="#3b82f6" stroke-width="2"></path>
  <path d="M5,5 Q20,5 30,20 Q20,35 5,35 Z" fill="white" stroke="#3b82f6" stroke-width="2"></path>
  <circle cx="5" cy="12" r="2" class="connection-point" data-type="input"></circle>
  <circle cx="5" cy="28" r="2" class="connection-point" data-type="input"></circle>
  <circle cx="35" cy="20" r="2" class="connection-point" data-type="output"></circle>
  `;
  connectionPoints = [
  { x: 5, y: 12, type: 'input' },
  { x: 5, y: 28, type: 'input' },
  { x: 35, y: 20, type: 'output' }
  ];
  break;
  case 'INPUT':
  svgContent = `
  <rect x="10" y="15" width="20" height="10" fill="white" stroke="#3b82f6" stroke-width="2" rx="2"></rect>
  <circle cx="35" cy="20" r="2" class="connection-point" data-type="output"></circle>
  <text x="20" y="23" font-size="8" text-anchor="middle" fill="#3b82f6">INPUT</text>
  <rect x="12" y="17" width="16" height="6" class="input-toggle" rx="1" fill="#e5e7eb" cursor="pointer"></rect>
  `;
  connectionPoints = [
  { x: 35, y: 20, type: 'output' }
  ];
  // Add click handler for input toggle
  setTimeout(() => {
  const inputToggle = gate.querySelector('.input-toggle');
  if (inputToggle) {
  inputToggle.addEventListener('click', function(e) {
  e.stopPropagation();
  const currentState = circuit.gates.get(gateId).state;
  circuit.setInputState(gateId, !currentState);
  this.setAttribute('fill', !currentState ? '#3b82f6' : '#e5e7eb');
  // Update connected components
  circuit.evaluate();
  updateOutputVisuals();
  });
  }
  }, 0);
  break;
  case 'OUTPUT':
  svgContent = `
  <rect x="10" y="15" width="20" height="10" fill="white" stroke="#3b82f6" stroke-width="2" rx="2"></rect>
  <circle cx="5" cy="20" r="2" class="connection-point" data-type="input"></circle>
  <text x="20" y="23" font-size="8" text-anchor="middle" fill="#3b82f6">OUTPUT</text>
  <rect x="12" y="17" width="16" height="6" class="output-indicator" rx="1" fill="#e5e7eb"></rect>
  `;
  connectionPoints = [
  { x: 5, y: 20, type: 'input' }
  ];
  break;
  }
  gate.innerHTML = svgContent;
  gate.setAttribute('data-connection-points', JSON.stringify(connectionPoints));
  gatesGroup.appendChild(gate);
  // Add event listeners for selection
  gate.addEventListener('mousedown', function(e) {
  if (currentTool === 'select') {
  e.stopPropagation();
  }
  });
  gate.addEventListener('contextmenu', function(e) {
  e.preventDefault();
  showContextMenu(e, gate);
  });
  return gate;
  }
  function createWire(start, end) {
  // Validate connection types
  if (start.point.type === end.point.type) {
  const errorMsg = document.createElement('div');
  errorMsg.className = 'fixed top-4 right-4 bg-red-100 border-l-4 border-red-500 text-red-700 p-4 rounded shadow-md z-50';
  errorMsg.innerHTML = `
  <div class="flex items-center">
  <div class="w-6 h-6 flex items-center justify-center mr-2">
  <i class="ri-error-warning-line"></i>
  </div>
  <p>Không thể kết nối hai điểm cùng loại (input-input hoặc output-output)</p>
  </div>`;
  document.body.appendChild(errorMsg);
  setTimeout(() => errorMsg.remove(), 3000);
  return null;
  }
  // Create logical wire in circuit
  const startGate = circuit.gates.get(start.element.id);
  const endGate = circuit.gates.get(end.element.id);
  const logicalWire = circuit.addWire(startGate, start.point, endGate, end.point);
  const wireId = logicalWire.id;
  const wire = document.createElementNS('http://www.w3.org/2000/svg', 'path');
  wire.setAttribute('id', wireId);
  wire.setAttribute('class', 'wire');
  wire.setAttribute('data-start-gate', start.element.id);
  wire.setAttribute('data-start-x', start.x);
  wire.setAttribute('data-start-y', start.y);
  wire.setAttribute('data-start-point-x', start.point.x);
  wire.setAttribute('data-start-point-y', start.point.y);
  wire.setAttribute('data-end-gate', end.element.id);
  wire.setAttribute('data-end-x', end.x);
  wire.setAttribute('data-end-y', end.y);
  wire.setAttribute('data-end-point-x', end.point.x);
  wire.setAttribute('data-end-point-y', end.point.y);
  const path = generatePath(start, end);
  wire.setAttribute('d', path);
  wiresGroup.appendChild(wire);
  return wire;
  }
  function generatePath(start, end) {
  const dx = end.x - start.x;
  const midX = start.x + dx / 2;
  return `M ${start.x} ${start.y} C ${midX} ${start.y}, ${midX} ${end.y}, ${end.x} ${end.y}`;
  }
  function findGateAtPosition(x, y) {
  const gates = document.querySelectorAll('#gates > g');
  for (let i = gates.length - 1; i >= 0; i--) {
  const gate = gates[i];
  const gateX = parseFloat(gate.getAttribute('data-x'));
  const gateY = parseFloat(gate.getAttribute('data-y'));
  // Simple bounding box check
  if (x >= gateX && x <= gateX + 40 && y >= gateY && y <= gateY + 40) {
  return gate;
  }
  }
  return null;
  }
  function findConnectionPointAtPosition(x, y, radius = 5) {
  const gates = document.querySelectorAll('#gates > g');
  for (let i = gates.length - 1; i >= 0; i--) {
  const gate = gates[i];
  const gateX = parseFloat(gate.getAttribute('data-x'));
  const gateY = parseFloat(gate.getAttribute('data-y'));
  const rotation = parseFloat(gate.getAttribute('data-rotation') || 0);
  const connectionPointsStr = gate.getAttribute('data-connection-points');
  if (!connectionPointsStr) continue;
  const connectionPoints = JSON.parse(connectionPointsStr);
  // Transform connection points based on rotation
  const transformedPoints = connectionPoints.map(point => {
  if (rotation === 0) return point;
  const centerX = 20;
  const centerY = 20;
  const rad = rotation * Math.PI / 180;
  const dx = point.x - centerX;
  const dy = point.y - centerY;
  return {
  x: centerX + dx * Math.cos(rad) - dy * Math.sin(rad),
  y: centerY + dx * Math.sin(rad) + dy * Math.cos(rad),
  type: point.type
  };
  });
  // Check if point already has a connection
  const existingConnections = Array.from(document.querySelectorAll('#wires > path')).filter(wire => {
  if (wire.getAttribute('data-end-gate') === gate.id) {
  const pointX = parseFloat(wire.getAttribute('data-end-point-x'));
  const pointY = parseFloat(wire.getAttribute('data-end-point-y'));
  return connectionPoints.some(p => p.x === pointX && p.y === pointY && p.type === 'input');
  }
  return false;
  });
  for (const point of transformedPoints) {
  const pointX = gateX + point.x;
  const pointY = gateY + point.y;
  const distance = Math.sqrt(Math.pow(x - pointX, 2) + Math.pow(y - pointY, 2));
  // Highlight connection point on hover
  const connectionPoint = gate.querySelector(`circle[cx="${point.x}"][cy="${point.y}"]`);
  if (distance <= radius && connectionPoint) {
  connectionPoint.setAttribute('fill', '#3b82f6');
  connectionPoint.setAttribute('r', '3');
  } else if (connectionPoint) {
  connectionPoint.setAttribute('fill', '#fff');
  connectionPoint.setAttribute('r', '2');
  }
  // Skip if input point already has a connection
  if (point.type === 'input' && existingConnections.some(wire => {
  const wirePointX = parseFloat(wire.getAttribute('data-end-point-x'));
  const wirePointY = parseFloat(wire.getAttribute('data-end-point-y'));
  return wirePointX === point.x && wirePointY === point.y;
  })) {
  continue;
  }
  if (distance <= radius) {
  return { gate, point };
  }
  }
  }
  return null;
  }
  function findWireAtPosition(x, y, threshold = 5) {
  const wires = document.querySelectorAll('#wires > path');
  for (let i = wires.length - 1; i >= 0; i--) {
  const wire = wires[i];
  // Check if point is near the wire
  if (isPointNearPath(wire, x, y, threshold)) {
  return wire;
  }
  }
  return null;
  }
  function isPointNearPath(path, x, y, threshold) {
  // Simplified check - in a real app, you'd need a more sophisticated algorithm
  const pathLength = path.getTotalLength();
  const steps = 50;
  const stepSize = pathLength / steps;
  for (let i = 0; i <= steps; i++) {
  const point = path.getPointAtLength(i * stepSize);
  const distance = Math.sqrt(Math.pow(x - point.x, 2) + Math.pow(y - point.y, 2));
  if (distance <= threshold) {
  return true;
  }
  }
  return false;
  }
  function updateConnectedWires(gate) {
  const gateId = gate.id;
  const gateX = parseFloat(gate.getAttribute('data-x'));
  const gateY = parseFloat(gate.getAttribute('data-y'));
  const rotation = parseFloat(gate.getAttribute('data-rotation') || 0);
  // Transform connection points based on rotation
  function transformPoint(point, rotation) {
  if (rotation === 0) return point;
  const centerX = 20;
  const centerY = 20;
  const rad = rotation * Math.PI / 180;
  const dx = point.x - centerX;
  const dy = point.y - centerY;
  return {
  x: centerX + dx * Math.cos(rad) - dy * Math.sin(rad),
  y: centerY + dx * Math.sin(rad) + dy * Math.cos(rad)
  };
  }
  // Update wires connected to this gate
  const connectedWires = document.querySelectorAll(`#wires > path[data-start-gate="${gateId}"], #wires > path[data-end-gate="${gateId}"]`);
  connectedWires.forEach(wire => {
  if (wire.getAttribute('data-start-gate') === gateId) {
  let pointX = parseFloat(wire.getAttribute('data-start-point-x'));
  let pointY = parseFloat(wire.getAttribute('data-start-point-y'));
  const transformedPoint = transformPoint({x: pointX, y: pointY}, rotation);
  pointX = transformedPoint.x;
  pointY = transformedPoint.y;
  wire.setAttribute('data-start-x', gateX + pointX);
  wire.setAttribute('data-start-y', gateY + pointY);
  const start = {
  x: gateX + pointX,
  y: gateY + pointY
  };
  const end = {
  x: parseFloat(wire.getAttribute('data-end-x')),
  y: parseFloat(wire.getAttribute('data-end-y'))
  };
  const path = generatePath(start, end);
  wire.setAttribute('d', path);
  }
  if (wire.getAttribute('data-end-gate') === gateId) {
  let pointX = parseFloat(wire.getAttribute('data-end-point-x'));
  let pointY = parseFloat(wire.getAttribute('data-end-point-y'));
  const transformedPoint = transformPoint({x: pointX, y: pointY}, rotation);
  pointX = transformedPoint.x;
  pointY = transformedPoint.y;
  wire.setAttribute('data-end-x', gateX + pointX);
  wire.setAttribute('data-end-y', gateY + pointY);
  const start = {
  x: parseFloat(wire.getAttribute('data-start-x')),
  y: parseFloat(wire.getAttribute('data-start-y'))
  };
  const end = {
  x: gateX + pointX,
  y: gateY + pointY
  };
  const path = generatePath(start, end);
  wire.setAttribute('d', path);
  }
  });
  }
  function removeGate(gate) {
  const gateId = gate.id;
  // Remove from circuit model
  circuit.removeGate(gateId);
  // Remove connected wires first
  const connectedWires = document.querySelectorAll(`#wires > path[data-start-gate="${gateId}"], #wires > path[data-end-gate="${gateId}"]`);
  connectedWires.forEach(wire => {
  wire.remove();
  });
  // Remove the gate
  gate.remove();
  // Update selection
  if (selectedElements.includes(gate)) {
  deselectElement(gate);
  }
  // Update circuit state
  circuit.evaluate();
  updateOutputVisuals();
  }
  function updateOutputVisuals() {
  // Update all output indicators
  document.querySelectorAll('#gates > g[data-gate-type="OUTPUT"]').forEach(outputGate => {
  const gateId = outputGate.id;
  const state = circuit.getOutputState(gateId);
  const indicator = outputGate.querySelector('.output-indicator');
  if (indicator) {
  indicator.setAttribute('fill', state ? '#3b82f6' : '#e5e7eb');
  }
  });
  // Update wire colors
  circuit.wires.forEach((wire, wireId) => {
  const wireElement = document.querySelector(`#wires > path#${wireId}`);
  if (wireElement) {
  wireElement.setAttribute('stroke', wire.state ? '#3b82f6' : '#94a3b8');
  }
  });
  }
  function removeWire(wire) {
  wire.remove();
  }
  function selectElement(element) {
  selectedElements.push(element);
  // Add selection indicator
  if (!element.querySelector('.selection-indicator')) {
  const rect = document.createElementNS('http://www.w3.org/2000/svg', 'rect');
  rect.setAttribute('class', 'selection-indicator');
  rect.setAttribute('x', '-2');
  rect.setAttribute('y', '-2');
  rect.setAttribute('width', '44');
  rect.setAttribute('height', '44');
  rect.setAttribute('fill', 'none');
  rect.setAttribute('stroke', '#3b82f6');
  rect.setAttribute('stroke-width', '2');
  rect.setAttribute('stroke-dasharray', '4,4');
  rect.setAttribute('pointer-events', 'none');
  element.insertBefore(rect, element.firstChild);
  }
  }
  function deselectElement(element) {
  const index = selectedElements.indexOf(element);
  if (index !== -1) {
  selectedElements.splice(index, 1);
  }
  // Remove selection indicator
  const indicator = element.querySelector('.selection-indicator');
  if (indicator) {
  indicator.remove();
  }
  }
  function clearSelection() {
  while (selectedElements.length > 0) {
  deselectElement(selectedElements[0]);
  }
  }
  // Context menu
  const contextMenu = document.getElementById('context-menu');
  function showContextMenu(e, element) {
  e.preventDefault();
  // Position the menu
  contextMenu.style.left = e.clientX + 'px';
  contextMenu.style.top = e.clientY + 'px';
  contextMenu.classList.remove('hidden');
  // Select the element if not already selected
  if (!selectedElements.includes(element)) {
  clearSelection();
  selectElement(element);
  }
  // Set up event handlers
  document.getElementById('rotate-element').onclick = function() {
  rotateSelectedElements();
  hideContextMenu();
  };
  document.getElementById('duplicate-element').onclick = function() {
  duplicateSelectedElements();
  hideContextMenu();
  };
  document.getElementById('delete-element').onclick = function() {
  deleteSelectedElements();
  hideContextMenu();
  };
  // Hide menu when clicking elsewhere
  document.addEventListener('click', hideContextMenuOnClick);
  }
  function hideContextMenu() {
  contextMenu.classList.add('hidden');
  document.removeEventListener('click', hideContextMenuOnClick);
  }
  function hideContextMenuOnClick(e) {
  if (!contextMenu.contains(e.target)) {
  hideContextMenu();
  }
  }
  function rotateSelectedElements() {
  // Simple 90-degree rotation
  selectedElements.forEach(element => {
  let currentRotation = element.getAttribute('data-rotation') || 0;
  currentRotation = parseInt(currentRotation);
  const newRotation = (currentRotation + 90) % 360;
  element.setAttribute('data-rotation', newRotation);
  // Apply rotation transform
  const x = parseFloat(element.getAttribute('data-x'));
  const y = parseFloat(element.getAttribute('data-y'));
  element.setAttribute('transform', `translate(${x}, ${y}) rotate(${newRotation}, 20, 20)`);
  // Update connected wires
  updateConnectedWires(element);
  });
  }
  function duplicateSelectedElements() {
  const newElements = [];
  selectedElements.forEach(element => {
  const gateType = element.getAttribute('data-gate-type');
  const x = parseFloat(element.getAttribute('data-x')) + gridSize;
  const y = parseFloat(element.getAttribute('data-y')) + gridSize;
  const newElement = createGate(gateType, x, y);
  newElements.push(newElement);
  });
  // Clear current selection and select new elements
  clearSelection();
  newElements.forEach(element => {
  selectElement(element);
  });
  }
  function deleteSelectedElements() {
  // Make a copy of the array since we'll be modifying it
  const elementsToDelete = [...selectedElements];
  elementsToDelete.forEach(element => {
  removeGate(element);
  });
  }
  // Initialize grid
  updateGridBackground();
  });
  </script>
  <script id="modal-interaction">
  document.addEventListener('DOMContentLoaded', function() {
  const modal = document.getElementById('modal');
  const saveBtn = document.getElementById('save-btn');
  const loadBtn = document.getElementById('load-btn');
  const closeModal = document.getElementById('close-modal');
  const saveForm = document.getElementById('save-form');
  const loadForm = document.getElementById('load-form');
  const modalTitle = document.getElementById('modal-title');
  const confirmSave = document.getElementById('confirm-save');
  const confirmLoad = document.getElementById('confirm-load');
  const clearBtn = document.getElementById('clear-btn');
  saveBtn.addEventListener('click', function() {
  modalTitle.textContent = 'Lưu thiết kế';
  saveForm.classList.remove('hidden');
  loadForm.classList.add('hidden');
  modal.classList.remove('hidden');
  });
  loadBtn.addEventListener('click', function() {
  modalTitle.textContent = 'Tải thiết kế';
  saveForm.classList.add('hidden');
  loadForm.classList.remove('hidden');
  modal.classList.remove('hidden');
  });
  closeModal.addEventListener('click', function() {
  modal.classList.add('hidden');
  });
  confirmSave.addEventListener('click', function() {
  const designName = document.getElementById('design-name').value;
  if (designName.trim() === '') {
  alert('Vui lòng nhập tên thiết kế');
  return;
  }
  // In a real app, you would save the circuit data here
  alert(`Đã lưu thiết kế "${designName}"`);
  modal.classList.add('hidden');
  });
  confirmLoad.addEventListener('click', function() {
  const designSelect = document.getElementById('saved-designs');
  const selectedDesign = designSelect.value;
  if (selectedDesign === '') {
  alert('Vui lòng chọn một thiết kế');
  return;
  }
  // In a real app, you would load the circuit data here
  alert(`Đã tải thiết kế "${designSelect.options[designSelect.selectedIndex].text}"`);
  modal.classList.add('hidden');
  });
  clearBtn.addEventListener('click', function() {
  if (confirm('Bạn có chắc chắn muốn xóa toàn bộ thiết kế hiện tại?')) {
  document.getElementById('gates').innerHTML = '';
  document.getElementById('wires').innerHTML = '';
  }
  });
  // Close modal when clicking outside
  modal.addEventListener('click', function(e) {
  if (e.target === modal) {
  modal.classList.add('hidden');
  }
  });
  });
  </script>
  </body>
  </html>
  ```
