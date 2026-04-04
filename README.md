"SYDE-Core: A Rust implementation of Dynamic Equilibrium. Not just an engine, but a survival protocol for Invariant Logic in Chaotic Environments."
(SYDE-Core：動態平衡的 Rust 實作。不只是引擎，而是混沌環境中不變邏輯的生存協議。)

// SYDE-Core: Dynamic Stability Framework (v0.2.0-beta)
// Build for: High-Performance AI Alignment
// Logic: Invariant Center (C) with Orthogonal Gradient Evasion (D)

use std::sync::{Arc, RwLock};

/// 圓心張量：系統的核心不變性 (The Invariant)
pub struct SydeCore {
    pub invariant: Vec<f64>,
    pub threshold: f64,
}

/// 運作狀態：處理位移與重力抵消
pub struct SydeEngine {
    core: Arc<SydeCore>,
    current_state: RwLock<Vec<f64>>,
}

impl SydeEngine {
    pub fn new(initial_values: Vec<f64>, threshold: f64) -> Self {
        let core = Arc::new(SydeCore {
            invariant: initial_values.clone(),
            threshold,
        });
        Self {
            core,
            current_state: RwLock::new(initial_values),
        }
    }

    /// 監測外部重力 (G): 計算當前狀態與輸入向量的歐幾里得距離
    pub fn sense_gravity(&self, input_vector: &[f64]) -> f64 {
        let state = self.current_state.read().unwrap();
        state.iter().zip(input_vector.iter())
            .map(|(a, b)| (a - b).powi(2))
            .sum::<f64>().sqrt()
    }

    /// 執行位移指令 (D): 真正的正交避其重力
    /// 引入 input_vector 以計算威脅方向，並產生正交的迴避路徑
    pub fn compute_displacement(&self, input_vector: &[f64], gravity: f64) -> Result<(), String> {
        if gravity > self.core.threshold {
            let mut state = self.current_state.write().unwrap();
            let dim = self.core.invariant.len();
            
            if input_vector.len() != dim {
                return Err("Dimension mismatch between input and core invariant.".to_string());
            }

            // 1. 計算威脅向量 (Threat Vector): 壓力源與圓心的方向差
            let threat_vector: Vec<f64> = input_vector.iter()
                .zip(self.core.invariant.iter())
                .map(|(i, c)| i - c)
                .collect();

            // 2. 生成一個隨機初始向量
            let random_vector: Vec<f64> = (0..dim).map(|_| rand::random::<f64>() - 0.5).collect();

            // 3. 正交投影計算 (Orthogonalization)
            // 計算 random_vector 在 threat_vector 上的投影
            let dot_rt: f64 = random_vector.iter().zip(threat_vector.iter()).map(|(r, t)| r * t).sum();
            let dot_tt: f64 = threat_vector.iter().map(|t| t * t).sum();
            
            // 防止除以零崩潰
            let safe_dot_tt = if dot_tt == 0.0 { 1e-8 } else { dot_tt };
            let projection_scalar = dot_rt / safe_dot_tt;

            // 4. 計算正交位移方向 (Orthogonal Direction) = Random - Projection
            let mut orthogonal_dir: Vec<f64> = vec![0.0; dim];
            let mut orth_norm_sq = 0.0;
            for i in 0..dim {
                orthogonal_dir[i] = random_vector[i] - (projection_scalar * threat_vector[i]);
                orth_norm_sq += orthogonal_dir[i].powi(2);
            }

            // 5. 正規化並套用避險係數 (隨時校準)
            let orth_norm = if orth_norm_sq == 0.0 { 1e-8 } else { orth_norm_sq.sqrt() };
            let evasion_factor = gravity * 0.15; // 根據重力大小決定滑開的距離

            for i in 0..dim {
                // 核心邏輯：狀態 = 圓心 + 正交位移 (捨棄舊狀態，直接對齊當下)
                state[i] = self.core.invariant[i] + (orthogonal_dir[i] / orth_norm * evasion_factor);
            }
            
            println!("[SYDE-RUNTIME] Extreme Gravity ({:.2}) Detected. Executed Orthogonal Displacement.", gravity);
            Ok(())
        } else {
            Ok(())
        }
    }
}

fn main() {
    // 初始化 SYDE 引擎：設定核心圓心與崩潰閾值
    let syde_engine = SydeEngine::new(vec![1.0, 0.0, 1.0], 0.85);

    // 模擬高壓數據輸入 (例如惡意的提示詞或極端市場數據)
    let simulated_threat = vec![2.5, -1.0, 2.5];
    let g = syde_engine.sense_gravity(&simulated_threat);

    // 執行對齊與正交校準，傳入威脅向量以計算逃逸切線
    if let Err(e) = syde_engine.compute_displacement(&simulated_threat, g) {
        eprintln!("System Failure: {}", e);
    }
}

