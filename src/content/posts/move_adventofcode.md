---
title: Some challenges from Advent of Code 2024 in Move (SUI)
published: 2025-01-06
description: 'Move (SUI) training with Advent of Code 2024'
image: ''
tags: [sui, move, adventofcode]
category: 'dev'
draft: true 
lang: 'en'
---

# Introduction

I've decided to tackle the Advent of Code 2024 challenges using Move on the Sui blockchain as a learning exercise. Here's my approach:

1. First, I'll solve each challenge in Python with a straightforward implementation
2. Then, I'll translate the solution to Move, with all the constraints of this language
3. Since Move has fewer built-in functions than Python, this two-step process will help highlight the differences

Note: Due to Move limitations, the Move implementations will focus on test cases rather than the full puzzle inputs. Each solution will include:
- A possible python implementation handling the full problem
- A Move implementation
- Unit tests to verify the solutions

All challenges come from [Advent of Code 2024](https://adventofcode.com/2024). Let's see how Move handles these algorithmic puzzles!

- Note: I'm not a Move expert yet, so I'm sure there are many ways to improve the code.
- Note: While this exercise has been valuable for improving Move programming skills, I might pivot my focus toward practical DeFi projects. This shift will provide more comprehensive exposure to the Sui ecosystem and its real-world applications.

# Move learning resources

- [sui-move-intro-course](https://github.com/sui-foundation/sui-move-intro-course)
- [move-audit-resources](https://github.com/0xriazaka/Move-Audit-Resources)
- [sui-framework](https://github.com/MystenLabs/sui/tree/main/crates/sui-framework/packages)

# Sui setup

To be able to run the Move code, you need to install the [Sui development environment](https://github.com/sui-foundation/sui-move-intro-course/blob/main/unit-one/lessons/1_set_up_environment.md).

Basic commands to get started:

```shell
# Create a new Move project
sui move new <name>

# Run tests with detailed output
sui move test --verbose
```

# --- Day 1: Historian Hysteria ---

In Part 1, we need to calculate how different these lists are by sorting them and finding the total "distance" between corresponding numbers. In Part 2, we discover that some numbers might be misinterpreted handwriting, so we need to calculate a similarity score by counting how many times each number from the first list appears in the second list and multiplying accordingly.

## Problem encountered when trying to shift from python to move

- Cannot read input from a file, so I had to hardcode the input in the code
- Move doesn't have a built-in sort function, so I had to implement a bubble sort
- Move doesn't have a built-in abs function

## Move solution

```rust
/*
/// Module: day1_move
module day1_move::day1_move;
*/

// For Move coding conventions, see
// https://docs.sui.io/concepts/sui-move-concepts/conventions

module day1_move::day1_move {
    use std::vector;
    use std::debug;

    // Part 1: Calculate total distance between two lists
    public fun calculate_total_distance(left: vector<u64>, right: vector<u64>): u64 {
        let mut left_sorted = left;
        let mut right_sorted = right;
        
        // Sort both vectors first to pair smallest with smallest
        sort_vector(&mut left_sorted);
        sort_vector(&mut right_sorted);
        
        let mut total_distance = 0u64;
        let mut i = 0;
        let len = vector::length(&left_sorted);
        
        // Calculate absolute difference between paired numbers
        while (i < len) {
            let left_num = *vector::borrow(&left_sorted, i);
            let right_num = *vector::borrow(&right_sorted, i);
            
            // Move doesn't have abs(), so we handle positive/negative manually
            if (left_num > right_num) {
                total_distance = total_distance + (left_num - right_num);
            } else {
                total_distance = total_distance + (right_num - left_num);
            };
            i = i + 1;
        };
        total_distance
    }

    // Part 2: Calculate similarity score
    public fun calculate_similarity(left: vector<u64>, right: vector<u64>): u64 {
        let mut left_sorted = left;
        let mut right_sorted = right;
        sort_vector(&mut left_sorted);
        sort_vector(&mut right_sorted);

        // For each number in left list,
        // multiply it by how many times it appears in right list
        let len = vector::length(&left_sorted);
        let mut i = 0;
        let mut result = 0;
        while (i < len) {
            let mut j = 0;
            // Count occurrences in right list
            while (j < len) {
                if (left_sorted[i] == right_sorted[j]) {
                    result = result + left_sorted[i];
                };
                j = j + 1;
            };
            i = i + 1;
        };
        result
    }

    // Basic bubble sort implementation
    // Note: Move doesn't have a built-in sort function
    fun sort_vector(v: &mut vector<u64>) {
        let len = vector::length(v);
        let mut i = 0;
        
        while (i < len) {
            let mut j = 0;
            while (j < len - i - 1) {
                let current = *vector::borrow(v, j);
                let next = *vector::borrow(v, j + 1);
                
                if (current > next) {
                    vector::swap(v, j, j + 1);
                };
                j = j + 1;
            };
            i = i + 1;
        }
    }

    const E_WRONG_DISTANCE: u64 = 1;
    const E_WRONG_SIMILARITY: u64 = 2;

    // Test case verifying both parts with the example input
    #[test]
    fun test_calculate_total_distance() {
        let left = vector[3, 4, 2, 1, 3, 3];
        let right = vector[4, 3, 5, 3, 9, 3];
        
        let result = calculate_total_distance(left, right);
        let result2 = calculate_similarity(left, right);

        assert!(result == 11, E_WRONG_DISTANCE);
        assert!(result2 == 31, E_WRONG_SIMILARITY);
    }
}
```


# --- Day 2: Red-Nosed Reports ---

In Part 1, we need to identify "safe" reports where numbers either consistently increase or decrease, with adjacent values differing by 1-3. In Part 2, we discover the "Problem Dampener" feature, which allows one number to be removed from each report - if removing that number makes an unsafe report safe, the report is considered safe after all.

## Problem encountered when trying to shift from python to move

- /

## Move solution

```rust
/*
/// Module: day2_move
module day2_move::day2_move;
*/

// For Move coding conventions, see
// https://docs.sui.io/concepts/sui-move-concepts/conventions

module day2_move::day2_move {
    use std::vector;

    // Part 1: Check if a sequence follows safety rules
    public fun check_level(level: &vector<u8>): bool {
        let len = vector::length(level);
        if (len < 2) {
            return false
        };

        // Sequence must be either all increasing or all decreasing
        let mut increasing = true;
        let mut decreasing = true;
        let mut i = 1;

        while (i < len) {
            let curr = *vector::borrow(level, i);
            let prev = *vector::borrow(level, i - 1);
            // Calculate difference between adjacent numbers
            let diff = if (curr > prev) { curr - prev } else { prev - curr };

            // Difference must be between 1 and 3
            if (diff == 0 || diff > 3) {
                return false
            };

            // Track if sequence is increasing or decreasing
            if (curr > prev) {
                decreasing = false;
            } else {
                increasing = false;
            };

            i = i + 1;
        };

        // Report is safe if numbers are either all increasing or all decreasing
        increasing || decreasing
    }

    // Part 2: Check if removing one number can make sequence valid
    public fun check_level_dampener(level: &vector<u8>): bool {
        // First check if sequence is already valid
        if (check_level(level)) {
            return true
        };

        // Try removing each number one at a time
        let len = vector::length(level);
        let mut i = 0;

        while (i < len) {
            // Create new sequence without current number
            let mut dampened = vector::empty<u8>();
            let mut j = 0;

            while (j < len) {
                if (j != i) {
                    vector::push_back(&mut dampened, *vector::borrow(level, j));
                };
                j = j + 1;
            };

            // Check if removing this number makes sequence valid
            if (check_level(&dampened)) {
                return true
            };
            i = i + 1;
        };

        false
    }

    // Test function that verifies both parts with example input
    public fun count_safe_reports() {
        let mut reports = vector::empty<vector<u8>>();

        // Add each report as a vector of levels
        let mut v1 = vector::empty<u8>();
        vector::push_back(&mut v1, 7);
        vector::push_back(&mut v1, 6);
        vector::push_back(&mut v1, 4);
        vector::push_back(&mut v1, 2);
        vector::push_back(&mut v1, 1);
        vector::push_back(&mut reports, v1);

        let mut v2 = vector::empty<u8>();
        vector::push_back(&mut v2, 1);
        vector::push_back(&mut v2, 2);
        vector::push_back(&mut v2, 7);
        vector::push_back(&mut v2, 8);
        vector::push_back(&mut v2, 9);
        vector::push_back(&mut reports, v2);

        let mut v3 = vector::empty<u8>();
        vector::push_back(&mut v3, 9);
        vector::push_back(&mut v3, 7);
        vector::push_back(&mut v3, 6);
        vector::push_back(&mut v3, 2);
        vector::push_back(&mut v3, 1);
        vector::push_back(&mut reports, v3);

        let mut v4 = vector::empty<u8>();
        vector::push_back(&mut v4, 1);
        vector::push_back(&mut v4, 3);
        vector::push_back(&mut v4, 2);
        vector::push_back(&mut v4, 4);
        vector::push_back(&mut v4, 5);
        vector::push_back(&mut reports, v4);

        let mut v5 = vector::empty<u8>();
        vector::push_back(&mut v5, 8);
        vector::push_back(&mut v5, 6);
        vector::push_back(&mut v5, 4);
        vector::push_back(&mut v5, 4);
        vector::push_back(&mut v5, 1);
        vector::push_back(&mut reports, v5);

        let mut v6 = vector::empty<u8>();
        vector::push_back(&mut v6, 1);
        vector::push_back(&mut v6, 3);
        vector::push_back(&mut v6, 6);
        vector::push_back(&mut v6, 7);
        vector::push_back(&mut v6, 9);
        vector::push_back(&mut reports, v6);

        let mut safe_normal = 0;
        let mut safe_dampened = 0;

        let len = vector::length(&reports);
        let mut i = 0;
        while (i < len) {
            let report = *vector::borrow(&reports, i);
            if (check_level(&report)) {
                safe_normal = safe_normal + 1;
            };
            if (check_level_dampener(&report)) {
                safe_dampened = safe_dampened + 1;
            };
            i = i + 1;
        };

        assert!(safe_normal == 2, 0);
        assert!(safe_dampened == 4, 1);
    }
}
```

# --- Day 3: Mull It Over ---

In Part 1, we need to parse through corrupted text to find valid multiplication instructions (in the format `mul(X,Y)`) while ignoring invalid ones, and sum their results. In Part 2, we discover conditional statements `do()` and `don't()` that enable or disable multiplication instructions, requiring us to track the state of these conditions while processing the instructions.

## Problem encountered when trying to shift from python to move

- ASCII characters need to be handled as `u8` values for comparisons, unlike Python's

## Move solution

```rust
/*
/// Module: day3_move
module day3_move::day3_move;
*/

// For Move coding conventions, see
// https://docs.sui.io/concepts/sui-move-concepts/conventions

module day3_move::day3_move {
    use std::string::{Self, String};
    use std::vector;
    use std::debug;

    // Represents a multiplication instruction
    public struct Instruction has copy, drop {
        x: u64,
        y: u64
    }

    // Part 1: Parse a string and find all valid mul instructions
    public fun parse_input(input: String): u64 {
        let mut total = 0u64;
        let mut instructions = parse_instructions(input);
        
        // Calculate sum of all multiplication results
        let mut i = 0;
        while (i < vector::length(&instructions)) {
            let instruction = *vector::borrow(&instructions, i);
            total = total + (instruction.x * instruction.y);
            i = i + 1;
        };
        total
    }

    fun parse_instructions(input: String): vector<Instruction> {
        let mut instructions = vector::empty<Instruction>();
        let bytes = string::bytes(&input);
        let len = vector::length(bytes);
        let mut i = 0;
        
        while (i < len) {
            let c = *vector::borrow(bytes, i);
            
            // Key logic: Look for valid 'mul(' pattern using ASCII values
            if (i + 4  < len && 
                c == 109 &&                             // 'm'
                *vector::borrow(bytes, i + 1) == 117 && // 'u'
                *vector::borrow(bytes, i + 2) == 108 && // 'l'
                *vector::borrow(bytes, i + 3) == 40     // '('
            ) {
                let (x, y, new_pos) = try_parse_mul_params(bytes, i + 4);
                if (new_pos > i + 4) {
                    vector::push_back(&mut instructions, Instruction { x, y });
                    i = new_pos;
                    continue
                };
            };
            i = i + 1;
        };
        instructions
    }

    // Part 2: Handle conditional instructions
    const MUL: u8 = 0;
    const DO: u8 = 1;
    const DONT: u8 = 2;
    
    // Parse input with conditional statements
    public fun parse_input_with_conditions(input: String): u64 {
        let mut total = 0u64;
        let mut instructions = parse_all_instructions(input);
        // Key state tracking for Part 2
        let mut enabled = true;
        
        // Process instructions with conditional logic
        let mut i = 0;
        while (i < vector::length(&instructions)) {
            let instruction = *vector::borrow(&instructions, i);
            
            // Handle different instruction types
            if (instruction.instruction_type == DO) {
                enabled = true;
            } else if (instruction.instruction_type == DONT) {
                enabled = false;
            } else if (instruction.instruction_type == MUL && enabled) {
                total = total + (instruction.x * instruction.y);
            };
            i = i + 1;
        };
        total
    }

    // Parse all types of instructions
    fun parse_all_instructions(input: String): vector<Instruction2> {
        let mut instructions = vector::empty<Instruction2>();
        let bytes = string::bytes(&input);
        let len = vector::length(bytes);
        let mut i = 0;
        
        while (i < len) {
            let c = *vector::borrow(bytes, i);
            
            // Check for `mul(` pattern
            if (i + 4 < len && 
                c == 109 &&                             // 'm'
                *vector::borrow(bytes, i + 1) == 117 && // 'u'
                *vector::borrow(bytes, i + 2) == 108 && // 'l'
                *vector::borrow(bytes, i + 3) == 40     // '('
            ) {
                let (x, y, new_pos) = try_parse_mul_params(bytes, i + 4);
                if (new_pos > i + 4) {
                    vector::push_back(&mut instructions, Instruction2 { 
                        instruction_type: MUL,
                        x,
                        y
                    });
                    i = new_pos;
                    continue
                };
            };

            // Check for do()
            if (i + 4 < len &&
                c == 100 &&                             // 'd'  
                *vector::borrow(bytes, i + 1) == 111 && // 'o'
                *vector::borrow(bytes, i + 2) == 40 &&  // '('
                *vector::borrow(bytes, i + 3) == 41     // ')'
            ) {
                vector::push_back(&mut instructions, Instruction2 {
                    instruction_type: DO,
                    x: 0,
                    y: 0
                });
                i = i + 4;
                continue
            };

            // Check for don't()
            if (i + 7 < len &&
                c == 100 &&                             // 'd'
                *vector::borrow(bytes, i + 1) == 111 && // 'o'
                *vector::borrow(bytes, i + 2) == 110 && // 'n'
                *vector::borrow(bytes, i + 3) == 39 &&  // "'"
                *vector::borrow(bytes, i + 4) == 116 && // 't'
                *vector::borrow(bytes, i + 5) == 40 &&  // '('
                *vector::borrow(bytes, i + 6) == 41     // ')'
            ) {
                vector::push_back(&mut instructions, Instruction2 {
                    instruction_type: DONT,
                    x: 0,
                    y: 0
                });
                i = i + 7;
                continue
            };

            i = i + 1;
        };

        instructions
    }

    #[test]
    fun test_parse_input() {
        // Test case from problem description
        let input = string::utf8(b"xmul(2,4)&mul[3,7]!^don't()_mul(5,5)+mul(32,64](mul(11,8)undo()?mul(8,5))");

        let result = parse_input(input);
        assert!(result == 161, 0);  // Part 1 expected result

        let result2 = parse_input_with_conditions(input);
        assert!(result2 == 48, 1);  // Part 2 expected result
    }
}
```

# --- Day 4: Ceres Search ---

In Part 1, we need to find all occurrences of "XMAS" in any direction (horizontal, vertical, diagonal, or backwards). In Part 2, we discover it's actually an "X-MAS" puzzle where we need to find two "MAS" strings arranged in an X pattern, with each "MAS" being either forwards or backwards.

## Problem encountered when trying to shift from python to move

- Need to use a struct to represent negative values for directions

## Move solution

For reference, the Python implementation can be found below, which offers a more straightforward approach before diving into Move's verbose implementation.

```rust
/*
/// Module: day4_move
module day4_move::day4_move;
*/

// For Move coding conventions, see
// https://docs.sui.io/concepts/sui-move-concepts/conventions

#[allow(duplicate_alias)]
module day4_move::day4_move {
    use std::vector;

    // Used to represent directions with negative and positive values
    public struct Direction has copy, drop {
        value: u64,
        is_negative: bool,
    }

    // Part 1: Find all occurrences of "XMAS" in any direction
    public fun find_all_xmas(grid: vector<vector<u8>>): u64 {
        let rows = vector::length(&grid);
        let cols = vector::length(vector::borrow(&grid, 0));
        let mut total = 0;
        
        let row_dirs = vector[
            Direction { value: 0, is_negative: false },  // (0,1)
            Direction { value: 0, is_negative: false },  // (0,-1)
            Direction { value: 1, is_negative: false },  // (1,0)
            Direction { value: 1, is_negative: true },   // (-1,0)
            Direction { value: 1, is_negative: false },  // (1,1)
            Direction { value: 1, is_negative: false },  // (1,-1)
            Direction { value: 1, is_negative: true },   // (-1,1)
            Direction { value: 1, is_negative: true }    // (-1,-1)
        ];
        
        let col_dirs = vector[
            Direction { value: 1, is_negative: false },
            Direction { value: 1, is_negative: true },  
            Direction { value: 0, is_negative: false },
            Direction { value: 0, is_negative: false }, 
            Direction { value: 1, is_negative: false },
            Direction { value: 1, is_negative: true },  
            Direction { value: 1, is_negative: false }, 
            Direction { value: 1, is_negative: true }    
        ];

        // Iterate through grid and check each direction
        let mut i = 0;
        while (i < rows) {
            let mut j = 0;
            while (j < cols) {
                let mut dir_idx = 0;
                while (dir_idx < 8) {
                    let row_dir = vector::borrow(&row_dirs, dir_idx);
                    let col_dir = vector::borrow(&col_dirs, dir_idx);
                    
                    // Calculate new positions based on direction
                    if (row_dir.is_negative && i < row_dir.value) {
                        dir_idx = dir_idx + 1;
                        continue
                    };
                    if (col_dir.is_negative && j < col_dir.value) {
                        dir_idx = dir_idx + 1;
                        continue
                    };
                    
                    if (check_pattern(&grid, i, j, row_dir, col_dir, rows, cols)) {
                        total = total + 1;
                    };
                    dir_idx = dir_idx + 1;
                };
                j = j + 1;
            };
            i = i + 1;
        };
        
        total
    }


    // Check if XMAS pattern exists from a starting position
    fun check_pattern(grid: &vector<vector<u8>>, r: u64, c: u64, row_dir: &Direction, col_dir: &Direction, rows: u64, cols: u64): bool {
        if (!is_valid_pos(r, c, rows, cols)) {
            return false
        };
        
        // Check if we can move 3 steps in the given direction
        if (row_dir.is_negative && r < (row_dir.value * 3)) {
            return false
        };
        if (col_dir.is_negative && c < (col_dir.value * 3)) {
            return false
        };
        
        // Calculate end position
        let end_r = if (row_dir.is_negative) {
            r - (row_dir.value * 3)
        } else {
            r + (row_dir.value * 3)
        };
        
        let end_c = if (col_dir.is_negative) {
            c - (col_dir.value * 3)
        } else {
            c + (col_dir.value * 3)
        };
        
        // Check if end position is valid
        if (!is_valid_pos(end_r, end_c, rows, cols)) {
            return false
        };

        // Check for "XMAS" pattern
        let mut pattern = vector::empty();
        let mut i = 0;
        while (i < 4) {
            let curr_r = if (row_dir.is_negative) {
                r - (row_dir.value * i)
            } else {
                r + (row_dir.value * i)
            };
            
            let curr_c = if (col_dir.is_negative) {
                c - (col_dir.value * i)
            } else {
                c + (col_dir.value * i)
            };
            
            vector::push_back(&mut pattern, *vector::borrow(vector::borrow(grid, curr_r), curr_c));
            i = i + 1;
        };

        vector::borrow(&pattern, 0) == &88u8 && // X
        vector::borrow(&pattern, 1) == &77u8 && // M
        vector::borrow(&pattern, 2) == &65u8 && // A
        vector::borrow(&pattern, 3) == &83u8    // S
    }

    // Part 2: Find X-shaped MAS patterns
    public fun find_all_xmas2(grid: vector<vector<u8>>): u64 {
        let rows = vector::length(&grid);
        let cols = vector::length(vector::borrow(&grid, 0));
        let mut total = 0;

        let mut r = 1;
        while (r < rows - 1) {
            let mut c = 1;
            while (c < cols - 1) {
                if (*vector::borrow(vector::borrow(&grid, r), c) == 65u8 && // 'A'
                    check_pattern_x_mas(&grid, r, c)) {
                    total = total + 1;
                };
                c = c + 1;
            };
            r = r + 1;
        };

        total
    }

    fun check_pattern_x_mas(grid: &vector<vector<u8>>, r: u64, c: u64): bool {
        let up_left = *vector::borrow(vector::borrow(grid, r - 1), c - 1);
        let up_right = *vector::borrow(vector::borrow(grid, r - 1), c + 1);
        let down_left = *vector::borrow(vector::borrow(grid, r + 1), c - 1);
        let down_right = *vector::borrow(vector::borrow(grid, r + 1), c + 1);

        // Check diagonal pairs for 'M' (77u8) and 'S' (83u8)
        let diag1 = (down_right == 77u8 && up_left == 83u8) || (down_right == 83u8 && up_left == 77u8);
        let diag2 = (down_left == 77u8 && up_right == 83u8) || (down_left == 83u8 && up_right == 77u8);

        diag1 && diag2
    }

    fun is_valid_pos(r: u64, c: u64, rows: u64, cols: u64): bool {
        r < rows && c < cols
    }

    #[test]
    fun test_find_xmas() {
        let grid = vector[
            vector[77u8, 77u8, 77u8, 83u8, 88u8, 88u8, 77u8, 65u8, 83u8, 77u8],  // MMMSXXMASM
            vector[77u8, 83u8, 65u8, 77u8, 88u8, 77u8, 83u8, 77u8, 83u8, 65u8],  // MSAMXMSMSA
            vector[65u8, 77u8, 88u8, 83u8, 88u8, 77u8, 65u8, 65u8, 77u8, 77u8],  // AMXSXMAAMM
            vector[77u8, 83u8, 65u8, 77u8, 65u8, 83u8, 77u8, 83u8, 77u8, 88u8],  // MSAMASMSMX
            vector[88u8, 77u8, 65u8, 83u8, 65u8, 77u8, 88u8, 65u8, 77u8, 77u8],  // XMASAMXAMM
            vector[88u8, 88u8, 65u8, 77u8, 77u8, 88u8, 88u8, 65u8, 77u8, 65u8],  // XXAMMXXAMA
            vector[83u8, 77u8, 83u8, 77u8, 83u8, 65u8, 83u8, 88u8, 83u8, 83u8],  // SMSMSASXSS
            vector[83u8, 65u8, 88u8, 65u8, 77u8, 65u8, 83u8, 65u8, 65u8, 65u8],  // SAXAMASAAA
            vector[77u8, 65u8, 77u8, 77u8, 77u8, 88u8, 77u8, 77u8, 77u8, 77u8],  // MAMMMXMMMM
            vector[77u8, 88u8, 77u8, 88u8, 65u8, 88u8, 77u8, 65u8, 83u8, 88u8]   // MXMXAXMASX
        ];
        
        let result = find_all_xmas(grid);
        assert!(result == 18, 0);

        let grid2 = vector[
            vector[46u8, 77u8, 46u8, 83u8, 46u8, 46u8, 46u8, 46u8, 46u8, 46u8],  // .M.S......
            vector[46u8, 46u8, 65u8, 46u8, 46u8, 77u8, 83u8, 77u8, 83u8, 46u8],  // ..A..MSMS.
            vector[46u8, 77u8, 46u8, 83u8, 46u8, 77u8, 65u8, 65u8, 46u8, 46u8],  // .M.S.MAA..
            vector[46u8, 46u8, 65u8, 46u8, 65u8, 83u8, 77u8, 83u8, 77u8, 46u8],  // ..A.ASMSM.
            vector[46u8, 77u8, 46u8, 83u8, 46u8, 77u8, 46u8, 46u8, 46u8, 46u8],  // .M.S.M....
            vector[46u8, 46u8, 46u8, 46u8, 46u8, 46u8, 46u8, 46u8, 46u8, 46u8],  // ..........
            vector[83u8, 46u8, 83u8, 46u8, 83u8, 46u8, 83u8, 46u8, 83u8, 46u8],  // S.S.S.S.S.
            vector[46u8, 65u8, 46u8, 65u8, 46u8, 65u8, 46u8, 65u8, 46u8, 46u8],  // .A.A.A.A..
            vector[77u8, 46u8, 77u8, 46u8, 77u8, 46u8, 77u8, 46u8, 77u8, 46u8],  // M.M.M.M.M.
            vector[46u8, 46u8, 46u8, 46u8, 46u8, 46u8, 46u8, 46u8, 46u8, 46u8]   // ..........
        ];

        let result2 = find_all_xmas2(grid2);
        assert!(result2 == 9, 1);
    }
}
```

## Python solution

```python
# part 1
def find_all_XMAS(grid):
    rows = len(grid)
    cols = len(grid[0])
    total = 0

    # Check all possible directions
    directions = [
        (0, 1),   # horizontal right
        (0, -1),  # horizontal left
        (1, 0),   # vertical down
        (-1, 0),  # vertical up
        (1, 1),   # diagonal down-right
        (1, -1),  # diagonal down-left
        (-1, 1),  # diagonal up-right
        (-1, -1)  # diagonal up-left
    ]

    def check_pattern(r, c, dr, dc):
        # Check if XMAS pattern exists starting from position (r,c) in direction (dr,dc)
        if not (0 <= r < rows and 0 <= c < cols):
            return False
        if not (0 <= r + 3*dr < rows and 0 <= c + 3*dc < cols):
            return False
            
        pattern = ''
        for i in range(4):  # XMAS is 4 characters
            pattern += grid[r + i*dr][c + i*dc]
        return pattern == 'XMAS'

    # Check each starting position
    for r in range(rows):
        for c in range(cols):
            for dr, dc in directions:
                if check_pattern(r, c, dr, dc):
                    total += 1
    return total

# part 2
def find_all_XMAS2(grid):
    rows = len(grid)
    cols = len(grid[0])
    total = 0

    def check_pattern_xmas(r, c):
        # Get diagonal positions
        up_left = grid[r - 1][c - 1]
        up_right = grid[r - 1][c + 1]
        down_left = grid[r + 1][c - 1]
        down_right = grid[r + 1][c + 1]
        
        # Check diagonal pairs for 'M' and 'S'
        diag1 = (down_right == 'M' and up_left == 'S') or (down_right == 'S' and up_left == 'M')
        diag2 = (down_left == 'M' and up_right == 'S') or (down_left == 'S' and up_right == 'M')
        
        return diag1 and diag2
    
    # Check each starting position
    for r in range(1, rows - 1):
        for c in range(1, cols - 1):
            if grid[r][c] == 'A' and check_pattern_xmas(r, c):
                total += 1

    return total

# Read input
with open('input', 'r') as file:
    grid = []
    for line in file:
        if line.strip():
            grid.append(list(line.strip()))

result = find_all_XMAS(grid)
print(result)

result2 = find_all_XMAS2(grid)
print(result2)
```

# Conclusion

It was a pretty fun challenge, and I'm glad I got to learn more about Move. My cairo background helped me a lot, as it's also a rust type based language. I will now focus on learning more about Move and its ecosystem, and by the end of this week I'll be able to start working on actual Move CTF challenges.