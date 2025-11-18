<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title></title>
  <!-- 引入Tailwind CSS -->
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- 引入Font Awesome -->
  <link href="https://cdn.jsdelivr.net/npm/font-awesome@4.7.0/css/font-awesome.min.css" rel="stylesheet">
  
  <!-- 配置Tailwind自定义样式 -->
  <script>
    tailwind.config = {
      theme: {
        extend: {
          colors: {
            primary: '#ff4d6d',
            secondary: '#7928ca',
          },
          fontFamily: {
            sans: ['Inter', 'system-ui', 'sans-serif'],
          },
        },
      }
    }
  </script>
  
  <style type="text/tailwindcss">
    @layer utilities {
      .content-auto {
        content-visibility: auto;
      }
      .text-shadow {
        text-shadow: 0 2px 10px rgba(255, 77, 109, 0.7);
      }
      .bg-gradient-love {
        background: linear-gradient(135deg, #ff4d6d 0%, #7928ca 100%);
      }
    }
  </style>
</head>
<body class="overflow-hidden bg-black min-h-screen font-sans text-white">
  <!-- 页面标题 -->
  <div class="fixed top-0 left-0 w-full p-6 text-center z-10">
    <h1 class="text-[clamp(1.5rem,3vw,2.5rem)] font-bold text-shadow animate-pulse">
      <i class="fa fa-heart text-primary mr-2 fa-rotate-180"></i>TO:
    </h1>
    <p class="text-gray-300 mt-2 text-[clamp(0.9rem,1.5vw,1rem)]">独属于李冬雨的烟花</p>
  </div>

  <!-- Canvas容器 -->
  <canvas id="fireworksCanvas" class="absolute top-0 left-0 w-full h-full"></canvas>

  <!-- 页脚提示 -->
  <div class="fixed bottom-6 left-0 w-full text-center z-10 text-gray-400 text-sm">
    <p>❤️ 希望李冬雨天天开心❤️</p>
  </div>

  <script>
    // 获取Canvas和上下文
    const canvas = document.getElementById('fireworksCanvas');
    const ctx = canvas.getContext('2d');
    
    // 设置Canvas尺寸为窗口大小
    function resizeCanvas() {
      canvas.width = window.innerWidth;
      canvas.height = window.innerHeight;
    }
    resizeCanvas();
    window.addEventListener('resize', resizeCanvas);
    
    // 粒子类
    class Particle {
      constructor(x, y, color) {
        this.x = x;
        this.y = y;
        this.color = color;
        this.velocity = {
          x: (Math.random() - 0.5) * 8,
          y: (Math.random() - 0.5) * 8
        };
        this.gravity = 0.1;
        this.alpha = 1;
        this.fade = 0.01;
        this.size = Math.random() * 3 + 1;
      }
      
      // 更新粒子状态
      update() {
        this.velocity.y += this.gravity;
        this.x += this.velocity.x;
        this.y += this.velocity.y;
        this.alpha -= this.fade;
      }
      
      // 绘制粒子
      draw() {
        ctx.save();
        ctx.globalAlpha = this.alpha;
        ctx.beginPath();
        ctx.arc(this.x, this.y, this.size, 0, Math.PI * 2);
        ctx.fillStyle = this.color;
        ctx.fill();
        
        // 添加光晕效果
        const gradient = ctx.createRadialGradient(
          this.x, this.y, 0,
          this.x, this.y, this.size * 2
        );
        gradient.addColorStop(0, this.color);
        gradient.addColorStop(1, 'rgba(255,255,255,0)');
        
        ctx.beginPath();
        ctx.arc(this.x, this.y, this.size * 2, 0, Math.PI * 2);
        ctx.fillStyle = gradient;
        ctx.fill();
        
        ctx.restore();
      }
    }
    
    // 倒置爱心烟花类
    class HeartFirework {
      constructor(x, y) {
        this.x = x;
        this.y = y;
        this.particles = [];
        this.exploded = false;
        this.timer = 0;
        this.maxTimer = 60; // 上升时间
        this.velocity = {
          x: 0,
          y: -8 - Math.random() * 2
        };
        this.color = this.getRandomColor();
        
        // 创建初始烟花弹
        this.fireworkSize = 4;
      }
      
      // 获取随机爱心色系
      getRandomColor() {
        const colors = [
          '#ff4d6d', '#ff758f', '#ff8fa3', '#ffb3c1',
          '#7928ca', '#8b5cf6', '#a78bfa', '#c4b5fd',
          '#ec4899', '#f472b6', '#f9a8d4', '#fccfe8'
        ];
        return colors[Math.floor(Math.random() * colors.length)];
      }
      
      // 生成倒置爱心形状的粒子位置（核心修改：将y值取反）
      createHeartParticles() {
        const particlesCount = 150; // 粒子数量
        const scale = 15 + Math.random() * 10; // 爱心大小
        
        for (let i = 0; i < particlesCount; i++) {
          const t = Math.random() * Math.PI * 2;
          // 爱心数学公式: (x²+y²-1)³ - x²y³ = 0
          // 核心修改：将y值乘以-1，实现爱心倒置
          const x = 16 * Math.pow(Math.sin(t), 3);
          const y = -(13 * Math.cos(t) - 5 * Math.cos(2*t) - 2 * Math.cos(3*t) - Math.cos(4*t));
          
          // 转换为实际坐标并添加随机偏移
          const px = this.x + x * scale + (Math.random() - 0.5) * 10;
          const py = this.y + y * scale + (Math.random() - 0.5) * 10;
          
          // 创建粒子
          const particle = new Particle(px, py, this.color);
          this.particles.push(particle);
        }
      }
      
      // 更新烟花状态
      update() {
        if (!this.exploded) {
          // 烟花上升阶段
          this.y += this.velocity.y;
          this.velocity.y += 0.1; // 重力作用
          this.timer++;
          
          // 到达最高点后爆炸
          if (this.timer >= this.maxTimer) {
            this.exploded = true;
            this.createHeartParticles();
          }
        } else {
          // 粒子扩散阶段
          for (let i = this.particles.length - 1; i >= 0; i--) {
            this.particles[i].update();
            
            // 移除透明度为0的粒子
            if (this.particles[i].alpha <= 0) {
              this.particles.splice(i, 1);
            }
          }
        }
      }
      
      // 绘制烟花
      draw() {
        if (!this.exploded) {
          // 绘制上升的烟花弹
          ctx.save();
          ctx.beginPath();
          ctx.arc(this.x, this.y, this.fireworkSize, 0, Math.PI * 2);
          
          // 渐变效果
          const gradient = ctx.createRadialGradient(
            this.x, this.y, 0,
            this.x, this.y, this.fireworkSize
          );
          gradient.addColorStop(0, 'white');
          gradient.addColorStop(1, this.color);
          
          ctx.fillStyle = gradient;
          ctx.fill();
          
          // 绘制尾迹
          ctx.beginPath();
          ctx.moveTo(this.x, this.y + this.fireworkSize);
          ctx.lineTo(this.x, this.y + this.fireworkSize + 20);
          ctx.strokeStyle = this.color;
          ctx.lineWidth = 2;
          ctx.stroke();
          ctx.restore();
        } else {
          // 绘制粒子
          this.particles.forEach(particle => particle.draw());
        }
      }
      
      // 检查烟花是否已结束
      isFinished() {
        return this.exploded && this.particles.length === 0;
      }
    }
    
    // 星空背景类
    class Star {
      constructor() {
        this.x = Math.random() * canvas.width;
        this.y = Math.random() * canvas.height;
        this.size = Math.random() * 2 + 0.5;
        this.alpha = Math.random() * 0.8 + 0.2;
        this.twinkleSpeed = Math.random() * 0.02 + 0.005;
        this.twinkleDirection = Math.random() > 0.5 ? 1 : -1;
      }
      
      update() {
        this.alpha += this.twinkleDirection * this.twinkleSpeed;
        
        if (this.alpha > 1) {
          this.alpha = 1;
          this.twinkleDirection = -1;
        } else if (this.alpha < 0.2) {
          this.alpha = 0.2;
          this.twinkleDirection = 1;
        }
      }
      
      draw() {
        ctx.save();
        ctx.globalAlpha = this.alpha;
        ctx.beginPath();
        ctx.arc(this.x, this.y, this.size, 0, Math.PI * 2);
        ctx.fillStyle = 'white';
        ctx.fill();
        ctx.restore();
      }
    }
    
    // 初始化星空
    const stars = [];
    function initStars() {
      const starCount = Math.min(Math.floor((canvas.width * canvas.height) / 10000), 200);
      for (let i = 0; i < starCount; i++) {
        stars.push(new Star());
      }
    }
    initStars();
    
    // 烟花数组
    const fireworks = [];
    
    // 鼠标点击事件 - 触发烟花
    canvas.addEventListener('click', (e) => {
      fireworks.push(new HeartFirework(e.clientX, e.clientY));
      
      // 添加点击效果
      ctx.beginPath();
      ctx.arc(e.clientX, e.clientY, 15, 0, Math.PI * 2);
      const gradient = ctx.createRadialGradient(
        e.clientX, e.clientY, 0,
        e.clientX, e.clientY, 15
      );
      gradient.addColorStop(0, 'rgba(255,255,255,0.8)');
      gradient.addColorStop(1, 'rgba(255,77,109,0)');
      ctx.fillStyle = gradient;
      ctx.fill();
    });
    
    // 自动发射烟花
    setInterval(() => {
      if (Math.random() > 0.7) {
        const x = Math.random() * canvas.width * 0.8 + canvas.width * 0.1;
        const y = Math.random() * canvas.height * 0.5 + canvas.height * 0.2;
        fireworks.push(new HeartFirework(x, y));
      }
    }, 3000);
    
    // 动画循环
    function animate() {
      // 半透明背景实现拖影效果
      ctx.fillStyle = 'rgba(0, 0, 0, 0.1)';
      ctx.fillRect(0, 0, canvas.width, canvas.height);
      
      // 更新和绘制星星
      stars.forEach(star => {
        star.update();
        star.draw();
      });
      
      // 更新和绘制烟花
      for (let i = fireworks.length - 1; i >= 0; i--) {
        fireworks[i].update();
        fireworks[i].draw();
        
        // 移除已结束的烟花
        if (fireworks[i].isFinished()) {
          fireworks.splice(i, 1);
        }
      }
      
      requestAnimationFrame(animate);
    }
    
    // 启动动画
    animate();
  </script>
</body>
</html>
