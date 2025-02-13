<!--
```agda
{-# OPTIONS --lossy-unification #-}
open import Cat.Monoidal.Instances.Cartesian
open import Cat.Displayed.Univalence.Thin
open import Cat.Instances.Sets.Complete
open import Cat.Displayed.Functor
open import Cat.Bi.Diagram.Monad
open import Cat.Displayed.Base
open import Cat.Displayed.Path
open import Cat.Monoidal.Base
open import Cat.Bi.Base
open import Cat.Prelude

import Algebra.Monoid.Category as Mon
import Algebra.Monoid as Mon

import Cat.Diagram.Monad as Mo
import Cat.Reasoning
```
-->

```agda
module Cat.Monoidal.Diagram.Monoid where
```

<!--
```agda
module _ {o ℓ} {C : Precategory o ℓ} (M : Monoidal-category C) where
  private module C where
    open Cat.Reasoning C public
    open Monoidal-category M public
```
-->

# Monoids in a monoidal category

Let $(\cC, \otimes, 1)$ be a [monoidal category] you want to study.
It can be, for instance, one of the [endomorphism categories] in a
[[bicategory]] that you like. A **monoid object in $\cC$**, generally
just called a "monoid in $\cC$", is really a collection of diagrams
in $\cC$ centered around an object $M$, the monoid itself.

[monoidal category]: Cat.Monoidal.Base.html#monoidal-categories
[endomorphism categories]: Cat.Monoidal.Base.html#endomorphism-categories

In addition to the object, we also require a "unit" map $\eta : 1 \to M$
and "multiplication" map $\mu : M \otimes M \to M$. Moreover, these maps
should be compatible with the unitors and associator of the underlying
monoidal category:

```agda
  record Monoid-on (M : C.Ob) : Type ℓ where
    no-eta-equality
    field
      η : C.Hom C.Unit M
      μ : C.Hom (M C.⊗ M) M

      μ-unitl : μ C.∘ (η C.⊗₁ C.id) ≡ C.λ←
      μ-unitr : μ C.∘ (C.id C.⊗₁ η) ≡ C.ρ←
      μ-assoc : μ C.∘ (C.id C.⊗₁ μ) ≡ μ C.∘ (μ C.⊗₁ C.id) C.∘ C.α← _ _ _
```

If we think of $\cC$ as a bicategory with a single object $*$ ---
that is, we _deloop_ it ---, then a monoid in $\cC$ is given by
precisely the same data as a monad in $\bf{B}\cC$, on the object $*$.

<!--
```agda
  private
    BC = Deloop M
    module BC = Prebicategory BC
  open Monoid-on

  Monoid-pathp
    : ∀ {P : I → C.Ob} {x : Monoid-on (P i0)} {y : Monoid-on (P i1)}
    → PathP (λ i → C.Hom C.Unit (P i)) (x .η) (y .η)
    → PathP (λ i → C.Hom (P i C.⊗ P i) (P i)) (x .μ) (y .μ)
    → PathP (λ i → Monoid-on (P i)) x y
  Monoid-pathp {x = x} {y} p q i .η = p i
  Monoid-pathp {x = x} {y} p q i .μ = q i
  Monoid-pathp {P = P} {x} {y} p q i .μ-unitl =
    is-prop→pathp
      (λ i → C.Hom-set _ (P i) (q i C.∘ (p i C.⊗₁ C.id)) C.λ←)
      (x .μ-unitl)
      (y .μ-unitl)
      i
  Monoid-pathp {P = P} {x} {y} p q i .μ-unitr =
    is-prop→pathp
      (λ i → C.Hom-set _ (P i) (q i C.∘ (C.id C.⊗₁ p i)) C.ρ←)
      (x .μ-unitr)
      (y .μ-unitr)
      i
  Monoid-pathp {P = P} {x} {y} p q i .μ-assoc =
    is-prop→pathp
      (λ i → C.Hom-set _ (P i)
        (q i C.∘ (C.id C.⊗₁ q i))
        (q i C.∘ (q i C.⊗₁ C.id) C.∘ C.α← _ _ _))
      (x .μ-assoc)
      (y .μ-assoc)
      i
```
-->

```agda
  monad→monoid : (M : Monad BC tt) → Monoid-on (M .Monad.M)
  monad→monoid M = go where
    module M = Monad M
    go : Monoid-on M.M
    go .η = M.η
    go .μ = M.μ
    go .μ-unitl = M.μ-unitl
    go .μ-unitr = M.μ-unitr
    go .μ-assoc = M.μ-assoc

  monoid→monad : ∀ {M} → Monoid-on M → Monad BC tt
  monoid→monad M = go where
    module M = Monoid-on M
    go : Monad BC tt
    go .Monad.M = _
    go .Monad.μ = M.μ
    go .Monad.η = M.η
    go .Monad.μ-assoc = M.μ-assoc
    go .Monad.μ-unitr = M.μ-unitr
    go .Monad.μ-unitl = M.μ-unitl
```

Put another way, a monad is just a monoid in the category of
~~endofunctors~~ endo-1-cells, what's the problem?

## The category Mon(C)

The monoid objects in $\cC$ can be made into a category, much like the
[monoids in the category of sets]. In fact, we shall see later that when
$\Sets$ is equipped with its [Cartesian monoidal structure],
$\rm{Mon}(\Sets) \cong \rm{Mon}$. Rather than defining $\rm{Mon}(\cC)$
directly as a category, we instead define it as a category
$\rm{Mon}(\cC) \liesover \cC$ [[displayed over|displayed category]]
$\cC$, which fits naturally with the way we have defined
`Monoid-object-on`{.Agda}.

[Cartesian monoidal structure]: Cat.Monoidal.Instances.Cartesian.html
[monoids in the category of sets]: Algebra.Monoid.html

<!--
```agda
module _ {o ℓ} {C : Precategory o ℓ} (M : Monoidal-category C) where
  private module C where
    open Cat.Reasoning C public
    open Monoidal-category M public
```
-->

Therefore, rather than defining a type of monoid homomorphisms, we
define a predicate on maps $f : m \to n$ expressing the condition of
being a monoid homomorphism. This is the familiar condition from
algebra, but expressed in a point-free way:

```agda
  record
    is-monoid-hom {m n} (f : C.Hom m n)
     (mo : Monoid-on M m) (no : Monoid-on M n) : Type ℓ where

    private
      module m = Monoid-on mo
      module n = Monoid-on no

    field
      pres-η : f C.∘ m.η ≡ n.η
      pres-μ : f C.∘ m.μ ≡ n.μ C.∘ (f C.⊗₁ f)
```

Since being a monoid homomorphism is a pair of propositions, the overall
condition is a proposition as well. This means that we will not need to
concern ourselves with proving displayed identity and associativity
laws, a great simplification.

<!--
```agda
  private unquoteDecl eqv = declare-record-iso eqv (quote is-monoid-hom)

  is-monoid-hom-is-prop : ∀ {m n} {f : C.Hom m n} {mo no} → is-prop (is-monoid-hom f mo no)
  is-monoid-hom-is-prop = is-hlevel≃ 1 (Iso→Equiv eqv) hlevel!

  open Displayed
  open Functor
  open is-monoid-hom
```
-->

```agda
  Mon[_] : Displayed C ℓ ℓ
  Mon[_] .Ob[_]  = Monoid-on M
  Mon[_] .Hom[_] = is-monoid-hom
  Mon[_] .Hom[_]-set f x y = is-prop→is-set is-monoid-hom-is-prop
```

The most complicated step of putting together the displayed category of
monoid objects is proving that monoid homomorphisms are closed under
composition. However, even in the point-free setting of an arbitrary
category $\cC$, the reasoning isn't _that_ painful:

```agda
  Mon[ .id′ ] .pres-η = C.idl _
  Mon[ .id′ ] .pres-μ = C.idl _ ∙ C.intror (C.-⊗- .F-id)

  Mon[_] ._∘′_ fh gh .pres-η = C.pullr (gh .pres-η) ∙ fh .pres-η
  Mon[_] ._∘′_ {x = x} {y} {z} {f} {g} fh gh .pres-μ =
    (f C.∘ g) C.∘ x .Monoid-on.μ                ≡⟨ C.pullr (gh .pres-μ) ⟩
    f C.∘ y .Monoid-on.μ C.∘ (g C.⊗₁ g)         ≡⟨ C.extendl (fh .pres-μ) ⟩
    Monoid-on.μ z C.∘ (f C.⊗₁ f) C.∘ (g C.⊗₁ g) ≡˘⟨ C.refl⟩∘⟨ C.-⊗- .F-∘ _ _ ⟩
    Monoid-on.μ z C.∘ (f C.∘ g C.⊗₁ f C.∘ g)    ∎

  Mon[_] .idr′ f = is-prop→pathp (λ i → is-monoid-hom-is-prop) _ _
  Mon[_] .idl′ f = is-prop→pathp (λ i → is-monoid-hom-is-prop) _ _
  Mon[_] .assoc′ f g h = is-prop→pathp (λ i → is-monoid-hom-is-prop) _ _
```

<!--
```agda
private
  Setsₓ : ∀ {ℓ} → Monoidal-category (Sets ℓ)
  Setsₓ = Cartesian-monoidal Sets-products Sets-terminal
  Mon : ∀ {ℓ} → Displayed (Sets ℓ) _ _
  Mon = Thin-structure-over (Mon.Monoid-structure _)
```
-->

Constructing this displayed category for the Cartesian monoidal
structure on the category of sets, we find that it is but a few
renamings away from the ordinary category of monoids-on-sets. The only
thing out of the ordinary about the proof below is that we can establish
the _displayed categories_ themselves are identical, so it is a trivial
step to show they induce identical^[thus isomorphic, thus equivalent]
[[total categories]].

```agda
Mon[Sets]≡Mon : ∀ {ℓ} → Mon[ Setsₓ ] ≡ Mon {ℓ}
Mon[Sets]≡Mon {ℓ} = Displayed-path F (λ a → is-iso→is-equiv (fiso a)) ff where
  open Displayed-functor
  open Monoid-on
  open is-monoid-hom

  open Mon.Monoid-hom
  open Mon.Monoid-on
```

The construction proceeds in three steps: First, put together a functor
(displayed over the identity) $\rm{Mon}(\cC) \to \thecat{Mon}$; Then,
prove that its action on objects ("step 2") and action on morphisms
("step 3") are independently equivalences of types. The characterisation
of paths of displayed categories will take care of turning this data
into an identification.

```agda
  F : Displayed-functor Mon[ Setsₓ ] Mon Id
  F .F₀′ o .identity = o .η (lift tt)
  F .F₀′ o ._⋆_ x y = o .μ (x , y)
  F .F₀′ o .has-is-monoid .Mon.has-is-semigroup =
    record { has-is-magma = record { has-is-set = hlevel! }
           ; associative  = o .μ-assoc $ₚ _
           }
  F .F₀′ o .has-is-monoid .Mon.idl = o .μ-unitl $ₚ _
  F .F₀′ o .has-is-monoid .Mon.idr = o .μ-unitr $ₚ _
  F .F₁′ wit .pres-id = wit .pres-η $ₚ _
  F .F₁′ wit .pres-⋆ x y = wit .pres-μ $ₚ _
  F .F-id′ = prop!
  F .F-∘′ = prop!

  open is-iso

  fiso : ∀ a → is-iso (F .F₀′ {a})
  fiso T .inv m .η _ = m .identity
  fiso T .inv m .μ (a , b) = m ._⋆_ a b
  fiso T .inv m .μ-unitl = funext λ _ → m .idl
  fiso T .inv m .μ-unitr = funext λ _ → m .idr
  fiso T .inv m .μ-assoc = funext λ _ → m .associative
  fiso T .rinv x = Mon.Monoid-structure _ .id-hom-unique
    (record { pres-id = refl ; pres-⋆ = λ _ _ → refl })
    (record { pres-id = refl ; pres-⋆ = λ _ _ → refl })
  fiso T .linv m = Monoid-pathp Setsₓ refl refl

  ff : ∀ {a b : Set _} {f : ∣ a ∣ → ∣ b ∣} {a′ b′}
     → is-equiv (F₁′ F {a} {b} {f} {a′} {b′})
  ff {a} {b} {f} {a′} {b′} =
    prop-ext (is-monoid-hom-is-prop Setsₓ) (hlevel 1)
             (λ z → F₁′ F z) invs .snd
    where
      invs : Mon.Monoid-hom (F .F₀′ a′) (F .F₀′ b′) f
           → is-monoid-hom Setsₓ f a′ b′
      invs m .pres-η = funext λ _ → m .pres-id
      invs m .pres-μ = funext λ _ → m .pres-⋆ _ _
```
