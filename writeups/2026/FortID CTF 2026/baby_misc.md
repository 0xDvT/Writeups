---
title: "Baby Misc"
author: "dax"
date: 2026-10-10
categories: ["Intro", "FortID CTF 2026"]
partial_solve: false
used_ai: false
---

## Challenge

> Heard you're a M4st3r H4ck3r, can you re**trie**ve the flag from this archive?

We're given a huge zip file (which took more than 3mins to unzip on my system!!) with hundreds of nested directories, each with a numerical name.

## Recon

The word "trie" is bolded in the challenge description, giving us a huge hint. A trie, also called a "prefix tree", is a tree-like data structure which you can read more about on [Wikipedia](https://en.wikipedia.org/wiki/Trie). It's now obvious that the directory structure is supposed to be a huge trie, with each directory name representing an ASCII character.

## Solution

1. I wrote the following Python script to convert the directory structure into a string. It stores all the unexplored directories in a queue, and prints out the ASCII representation when we reach a terminal node (in this case, a file)

```python
import os

# ASCII for "FortID{", which we know is the beginning of the flag
base_path = './retrieve/70/111/114/116/73/68/123'
q = os.listdir(base_path)

while len(q) > 0:
    top = q.pop(0)
    if not os.path.isdir(f'{base_path}/{top}'):
        txt = "".join([chr(int(x)) for x in top.split('/')[:-1]])
        if "}" in txt:
            print(f"FortID{{{txt}") # the first 2 {'s are to print a literal "{" character, the next one is the format specifier, don't you love python escaping? :p
        continue
    next = os.listdir(f'{base_path}/{top}')
    next_list = [f'{top}/{x}' for x in next]
    q.extend([f'{top}/{x}' for x in next])
```

2. I expected this to only find one flag, but that would be too easy. We instead see 770 total flags (!!), most of them being randomly generated l33t speak non-sense. Not to worry, the last and longest flag (`FortID{not_the_right_flag_but_the_one_you_are_looking_for_is_similar}`) gives us a hint. Most of the flags have a `!` or `x` at the end, but not the last one, suggesting that the real flag won't either. I piped the output of the previous script into `flags.txt` and wrote another basic python script to filter the flags:

```python
with open('flags.txt') as h:
    flags = h.read().splitlines()

print("\n".join([x for x in flags if x[-1] == '}']))
```

Output:

```text
FortID{nice_try_now_keep_retrieving}
FortID{sudo_make_me_the_actual_flag}
FortID{D0_or_D0_N07_Th3r3_15_n0_Tri3}
FortID{there_is_no_flag_here_only_zuul}
FortID{plot_twist_this_one_is_also_fake}
FortID{wrong_tree_right_idea_keep_going}
FortID{congratulations_you_found_a_decoy}
FortID{grep_harder_the_truth_is_out_there}
FortID{one_does_not_simply_guess_the_flag}
FortID{all_your_flags_are_belong_to_decoy}
FortID{ctrl_f_will_not_save_you_this_time}
FortID{404_flag_not_found_please_try_again}
FortID{move_along_nothing_to_retrieve_here}
FortID{the_flag_is_a_lie_but_the_png_is_not}
FortID{totally_not_a_honeypot_pinky_promise}
FortID{your_princess_is_in_another_directory}
FortID{close_enough_only_counts_in_horseshoes}
FortID{you_had_one_job_and_it_was_not_this_one}
FortID{definitely_prod_credentials_do_not_share}
FortID{free_flag_no_clickbait_100_percent_legit}
FortID{this_is_definitely_the_flag_trust_me_bro}
FortID{flag_loading_please_insert_another_floppy}
FortID{works_on_my_machine_not_on_the_scoreboard}
FortID{achievement_unlocked_found_the_wrong_flag}
FortID{this_flag_has_been_moved_to_another_castle}
FortID{if_you_can_read_this_you_are_getting_warmer}
FortID{these_are_not_the_bytes_you_are_looking_for}
FortID{wrong_answer_but_excellent_directory_traversal}
FortID{thank_you_mario_but_your_flag_is_in_another_trie}
FortID{look_mom_I_found_a_flag_but_the_server_rejects_it}
FortID{please_ignore_the_suspiciously_flag_shaped_string}
FortID{the_real_flag_was_the_friends_we_made_along_the_way}
FortID{not_the_right_flag_but_the_one_you_are_looking_for_is_similar}
```

## Flag

`FortID{D0_or_D0_N07_Th3r3_15_n0_Tri3}`
