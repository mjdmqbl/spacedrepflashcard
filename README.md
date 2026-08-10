# spacedrepflashcard
import json
import os
import random
import sys
from datetime import datetime, timedelta

DATA_FILE = "flashcards.json"
BOX_INTERVALS = {1: 0, 2: 1, 3: 3, 4: 7, 5: 14}  # Days delay per box level

class Colors:
    GREEN = "\033[92m"
    RED = "\033[91m"
    YELLOW = "\033[93m"
    CYAN = "\033[96m"
    BOLD = "\033[1m"
    RESET = "\033[0m"

def load_deck():
    """Loads existing card deck from storage or initializes sample data."""
    if os.path.exists(DATA_FILE):
        with open(DATA_FILE, "r", encoding="utf-8") as f:
            return json.load(f)
    return [
        {"id": 1, "front": "What is Python?", "back": "An interpreted programming language.", "box": 1, "next_review": str(datetime.now().date())},
        {"id": 2, "front": "What is Big-O notation?", "back": "Algorithm complexity measurement.", "box": 1, "next_review": str(datetime.now().date())}
    ]

def save_deck(deck):
    """Saves deck state to local file."""
    with open(DATA_FILE, "w", encoding="utf-8") as f:
        json.dump(deck, f, indent=2)

def add_card(deck, front, back):
    """Creates a new flashcard and stores it in Box 1."""
    new_id = max([c["id"] for c in deck], default=0) + 1
    deck.append({
        "id": new_id,
        "front": front,
        "back": back,
        "box": 1,
        "next_review": str(datetime.now().date())
    })
    save_deck(deck)
    print(f"{Colors.GREEN}✔ Created Card #{new_id}{Colors.RESET}")

def review_deck(deck):
    """Runs interactive Leitner system review for cards due today."""
    today = str(datetime.now().date())
    due_cards = [c for c in deck if c["next_review"] <= today]

    if not due_cards:
        print(f"{Colors.GREEN}🎉 All caught up! No cards due for review today.{Colors.RESET}")
        return

    random.shuffle(due_cards)
    print(f"\n{Colors.BOLD}{Colors.CYAN}--- Flashcard Review ({len(due_cards)} due) ---{Colors.RESET}\n")

    for card in due_cards:
        print(f"{Colors.BOLD}Box {card['box']} | Prompt:{Colors.RESET} {card['front']}")
        input("Press [Enter] to reveal answer...")
        print(f"{Colors.YELLOW}Answer:{Colors.RESET} {card['back']}\n")

        while True:
            choice = input("Did you answer correctly? (y/n): ").strip().lower()
            if choice in ['y', 'n']:
                break

        if choice == 'y':
            card['box'] = min(5, card['box'] + 1)
            print(f"{Colors.GREEN}✔ Correct! Promoted to Box {card['box']}{Colors.RESET}\n")
        else:
            card['box'] = 1
            print(f"{Colors.RED}✘ Incorrect. Reset to Box 1{Colors.RESET}\n")

        days = BOX_INTERVALS[card['box']]
        card['next_review'] = str((datetime.now() + timedelta(days=days)).date())
        save_deck(deck)

def list_cards(deck):
    """Displays stored cards and current progression status."""
    print(f"\n{Colors.BOLD}{'ID':<5} {'Box':<5} {'Next Review':<12} {'Prompt':<30}{Colors.RESET}")
    print("-" * 55)
    for c in deck:
        prompt = c['front'] if len(c['front']) <= 27 else c['front'][:24] + "..."
        print(f"{c['id']:<5} {c['box']:<5} {c['next_review']:<12} {prompt:<30}")
    print()

def print_help():
    """Prints usage instructions."""
    print(f"{Colors.BOLD}Spaced Repetition Flashcard Engine{Colors.RESET}")
    print("Commands:")
    print("  python flashcards.py review             - Review cards due today")
    print("  python flashcards.py add \"Q\" \"A\"        - Add a new card")
    print("  python flashcards.py list               - List all cards & review dates")

def main():
    deck = load_deck()
    args = sys.argv[1:]

    if not args or args[0] == "review":
        review_deck(deck)
    elif args[0] == "add" and len(args) == 3:
        add_card(deck, args[1], args[2])
    elif args[0] == "list":
        list_cards(deck)
    else:
        print_help()

if __name__ == "__main__":
    main()
