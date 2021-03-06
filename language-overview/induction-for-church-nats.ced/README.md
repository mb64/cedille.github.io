# induction-for-church-nats.ced

```
module Nat.

-- define version of naturals via non-dependent elimination
Nat' ◂ ★ = ∀ X : ★ . (X ➔ X) ➔ X ➔ X .
zero' ◂ Nat' = Λ X . λ s . λ z . z .
suc' ◂ Nat' ➔ Nat' = λ n . Λ X . λ s . λ z . s (n · X s z) .

-- now describe dependent elimination
InductiveNat ◂ Nat' ➔ ★ =
  λ x : Nat' . ∀ Q : Nat' ➔ ★ .
  (∀ x : Nat' . Q x ➔ Q (suc' x)) ➔
  Q zero' ➔
  Q x.

-- a Nat is a Nat' that also supports dependent elimination
Nat ◂ ★ = ι x : Nat' . InductiveNat x.

-- now we can define the constructors for Nat
zero ◂ Nat = [ zero' , Λ X . λ s . λ z . z ] .
suc ◂ Nat ➔ Nat = λ n . [ suc' n.1 , Λ P . λ s . λ z . s -n.1 (n.2 · P s z) ] .

{- this function converts from Nat' to Nat just by using the
   Nat constructors in an elimination of a Nat' -}
toNat ◂ Nat' ➔ Nat = λ x . x · Nat suc zero .

-- we can prove that reflection is the identity on the erasures of the terms
reflectionNat ◂ Π n : Nat . { toNat n.1 ≃ n } =
  λ n . n.2 · (λ x : Nat' . {(toNat x).1 ≃ x}) (Λ x . λ ih . χ {suc' (toNat x).1 ≃ suc' x} - ρ ih - β) β .

{- now we prove induction for Nat-predicate P by a dependent elimination
   on Nat n.  sucince Nats only support induction on Nat'-predicates, we
   cannot use P directly.  Instead, we use the predicate

   λ x : Nat' . P (toNat x)

   and then use reflection to get rid of the reference to toNat. -}
inductionNat ◂ Π n : Nat . ∀ P : Nat ➔ ★ . (∀ m : Nat . P m ➔ P (suc m)) ➔ P zero ➔ P n =
  λ n . Λ P . λ s . λ z . 
    ρ (ς (reflectionNat n)) -
      (n.2 · (λ x : Nat' . P (toNat x))
        (Λ p . λ q . s -(toNat p) q) z) .
      
recNat ◂ Π n : Nat . ∀ X : ★ . (X ➔ X) ➔ X ➔ X = λ n . n.1.

add ◂ Nat ➔ Nat ➔ Nat =
  λ n . λ m . recNat n suc m.
```
