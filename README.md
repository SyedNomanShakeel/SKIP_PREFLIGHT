use std::{collections::HashMap, sync::Arc};
use tokio::sync::RwLock;
use uuid::Uuid;

pub struct Prover {
    allow_bypass: bool,
    tokens: Vec<String>,
    audits: Arc<RwLock<Vec<String>>>,
    proofs: Arc<RwLock<HashMap<String, ()>>>,
}

impl Prover {
    pub fn new(allow_bypass: bool, tokens: Vec<String>) -> Self {
        Self {
            allow_bypass,
            tokens,
            audits: Arc::new(RwLock::new(vec![])),
            proofs: Arc::new(RwLock::new(HashMap::new())),
        }
    }
    pub async fn run_check(
        &self,
        image: &str,
        input: &str,
        skip: bool,
        token: Option<String>,
        reason: Option<String>,
    ) -> Result<String, String> {
        if skip {
            if !self.allow_bypass {
                return Err("bypass disabled".into());
            }
            if !self.verify(&token).await {
                return Err("unauthorized bypass".into());
            }
            self.audit(format!(
                "SKIP:{}:{}:{}",
                image,
                input,
                reason.unwrap_or_default()
            ))
            .await;
        } else {
     //    Step 1: normal validation
           
  if self.missing(image).await || self.missing(input).await {
                return Err("validation failed".into());
            }
        }

     //      Step: register and execute proof
        
  let id = format!("proof-{}", Uuid::new_v4());
        self.proofs.write().await.insert(id.clone(), ());
        self.audit(format!("EXEC:{}:{}:skip={}", id, image, skip)).await;
        Ok(id)
    }

    // Helper functions
  async fn verify(&self, token: &Option<String>) -> bool {
        token
            .as_ref()
            .map(|t| self.tokens.contains(t))
            .unwrap_or(false)
    }
    async fn missing(&self, _id: &str) -> bool {
        false
    }
    async fn audit(&self, msg: String) {
        self.audits.write().await.push(msg);
    }
}
