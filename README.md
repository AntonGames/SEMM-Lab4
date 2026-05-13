# Lab 4 — Z Specification Language: Library Management System

## Domain

The specification models a **Library Management System** — a system for managing book collections, member registrations, borrowing/returning books, and searching the catalog.

Core entities:
- **Books** — identified by ISBN, with title and genre
- **Members** — registered library patrons (PERSON)
- **Loans** — active book borrowings (book → member mapping)
- **Waitlists** — reservation queues for books
- **Genres** — fiction, nonfiction, science, history, biography

## Tool

**CZT (Community Z Tools) 1.6** — a Java-based toolset for parsing, typechecking, and animating Z specifications.

- CLI: `java -jar czt/czt.jar library-z-spec.tex`
- Animator: `java -jar czt/czt.jar zlive` → `load library-z-spec.tex`

## Project Structure

| File / Directory | Description |
|---|---|
| `library-z-spec.tex` | Main Z specification in LaTeX markup |
| `README.md` | This file |
| `task.txt` | Lab assignment description |
| `czt/czt.jar` | CZT standalone tool (Java) |
| `Slides/` | Reference materials (slides, book, tool links) |

## Z Specification Overview

### Basic Types and Free Types
| Type | Description |
|---|---|
| `[PERSON, ISBN, TITLE]` | Given sets for domain entities |
| `GENRE` | Free type: fiction, nonfiction, science, history, biography |
| `REPORT` | Free type: operation result codes (ok, bookNotFound, etc.) |

### State Variables (6)
| Variable | Type | Description |
|---|---|---|
| `books` | `ℙ ISBN` | Set of all book ISBNs |
| `members` | `ℙ PERSON` | Set of registered members |
| `title` | `ISBN ⇸ TITLE` | Book title mapping |
| `genre` | `ISBN ⇸ GENRE` | Book genre mapping |
| `loans` | `ISBN ⇸ PERSON` | Active loan records |
| `waitlist` | `ISBN ⇸ seq PERSON` | Book waitlists |

### Invariants (6)
| # | Invariant |
|---|---|
| 1 | `dom title = books` — every book has a title |
| 2 | `dom genre = books` — every book has a genre |
| 3 | `dom loans ⊆ books` — only existing books can be loaned |
| 4 | `ran loans ⊆ members` — only members can borrow |
| 5 | `dom waitlist ⊆ books` — waitlists for existing books only |
| 6 | `∀ m : members • #loans⁻¹(m) ≤ maxLoans` — loan limit enforced |

### Functions (4)
| Function | Signature | Description |
|---|---|---|
| `borrowedBy` | `PERSON × loans → ℙ ISBN` | Books borrowed by a member |
| `availableBooks` | `books × loans → ℙ ISBN` | Books not currently on loan |
| `loanCount` | `PERSON × loans → ℕ` | Number of active loans for a member |
| `booksByGenre` | `GENRE × books × genre → ℙ ISBN` | Books filtered by genre |

### Operations (8, all with pre/post conditions)
| Operation | Type | Description |
|---|---|---|
| `AddBook` | Δ Library | Add a new book with title and genre |
| `RemoveBook` | Δ Library | Remove a book (must not be on loan) |
| `RegisterMember` | Δ Library | Register a new library member |
| `RemoveMember` | Δ Library | Remove a member (must have no loans) |
| `BorrowBook` | Δ Library | Loan a book to a member |
| `ReturnBook` | Δ Library | Return a borrowed book |
| **`SearchByGenre`** | Ξ Library | **Returns a set** of books by genre |
| **`GetMemberBooks`** | Ξ Library | **Returns a set** of books borrowed by member |

### Schema Calculus (4 schemas using operators)
| Schema | Operator | Description |
|---|---|---|
| `AddBookOk` | **∧ (conjunction)** | Combines AddBook with Success report |
| `TotalAddBook` | **∨ (disjunction)** | AddBookOk or BookAlreadyExistsError |
| `BookNotOnLoan` | **¬ (negation)** | Negation of BookOnLoan schema |
| `BorrowAndQuery` | **⨟ (composition)** | BorrowBook then GetMemberBooks |

### Data Refinement (3 refined operations)
Abstract (set-based `Library`) → Concrete (sequence-based `ConcreteLibrary`)

| Abstract | Concrete | Refinement |
|---|---|---|
| `BorrowBook` | `ConcreteBorrowBook` | Set union → sequence append |
| `ReturnBook` | `ConcreteReturnBook` | Domain subtraction → sequence filter |
| `SearchByGenre` | `ConcreteSearchByGenre` | Set comprehension → sequence iteration |

**Retrieve relation** maps concrete sequences back to abstract sets via `ran`.

## Requirements Coverage

| # | Requirement | Minimum | Actual |
|---|---|---|---|
| 1 | State variables | 5 | **6** ✓ |
| 2 | Functions | 3 | **4** ✓ |
| 3 | Invariants | 5 | **6** ✓ |
| 4 | Operations with pre+post conditions | 5 | **8** ✓ |
| 5 | Operations returning a set | 2 | **2** ✓ |
| 6 | Schemas via schema calculus operators | 3 | **4** ✓ |
| 7 | Refined operations with abstraction relation | 3 | **3** ✓ |

## How to Run

### Prerequisites
- Java JRE 8+ (tested with OpenJDK 21)

### Typecheck the specification
```bash
cd Lab4
java -jar czt/czt.jar library-z-spec.tex
```

Expected output: `0 warnings and errors`

### Load into ZLive (interactive)
```bash
java -jar czt/czt.jar zlive
```
Then in the ZLive REPL:
```
load library-z-spec.tex
```

> **Note:** ZLive animation (`do InitLibrary`) will not work because the specification uses abstract given sets (`ISBN`, `PERSON`, `TITLE`) which are infinite. ZLive can only animate schemas with finite, bounded types. The typechecker confirmation (`0 warnings and errors`) is the primary verification method.

### Compile to PDF
The included `library-z-spec.pdf` was compiled using **MiKTeX 25.12** (`pdflatex`) with the `oz` package on Windows:
```bash
pdflatex library-z-spec.tex
```
To install MiKTeX: `winget install MiKTeX.MiKTeX`

## Z Notation — LaTeX Commands Reference

All Z symbols in the specification are entered as plain ASCII LaTeX commands:

| LaTeX Command | Z Symbol | Meaning |
|---|---|---|
| `\power` | ℙ | Power set |
| `\pfun` | ⇸ | Partial function |
| `\fun` | → | Total function |
| `\mapsto` | ↦ | Maplet |
| `\nat` | ℕ | Natural numbers |
| `\nat_1` | ℕ₁ | Positive naturals |
| `\forall` | ∀ | Universal quantifier |
| `\exists` | ∃ | Existential quantifier |
| `\in` | ∈ | Membership |
| `\notin` | ∉ | Non-membership |
| `\subseteq` | ⊆ | Subset |
| `\emptyset` | ∅ | Empty set |
| `\dom` | dom | Domain |
| `\ran` | ran | Range |
| `\oplus` | ⊕ | Function override |
| `\ndres` | ⩤ | Domain subtraction |
| `\cup` | ∪ | Set union |
| `\cap` | ∩ | Set intersection |
| `\setminus` | \ | Set difference |
| `\langle \rangle` | ⟨ ⟩ | Sequence brackets |
| `\cat` | ⌢ | Sequence concatenation |
| `\Delta` | Δ | State change (includes before+after) |
| `\Xi` | Ξ | No state change |
| `\land` | ∧ | Schema conjunction |
| `\lor` | ∨ | Schema disjunction |
| `\lnot` | ¬ | Schema negation |
| `\semi` | ⨟ | Schema composition |
| `\leq` | ≤ | Less or equal |
| `\geq` | ≥ | Greater or equal |
| `\neq` | ≠ | Not equal |
| `\#` | # | Cardinality |
| `\\` | | Line separator in schemas |
| `\where` | | Separates declarations from predicates |

No special keyboard or fonts are needed — everything is typed in a plain text editor.

## References

- J.M. Spivey, *The Z Notation: A Reference Manual*, 2nd ed., 2001
- J. Woodcock & J. Davies, *Using Z: Specification, Refinement, and Proof*, 1996
- CZT: https://czt.sourceforge.net/
- Z Word Tools: https://zwordtools.sourceforge.net/
- Lecture slides: `Slides/Z-system-slides.pdf`
