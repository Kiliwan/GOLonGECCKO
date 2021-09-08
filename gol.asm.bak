    ;;    game state memory location
    .equ CURR_STATE, 0x1000              ; current game state
    .equ GSA_ID, 0x1004                     ; gsa currently in use for drawing
    .equ PAUSE, 0x1008                     ; is the game paused or running
    .equ SPEED, 0x100C                      ; game speed
    .equ CURR_STEP,  0x1010              ; game current step
    .equ SEED, 0x1014              ; game seed
    .equ GSA0, 0x1018              ; GSA0 starting address
    .equ GSA1, 0x1038              ; GSA1 starting address
    .equ SEVEN_SEGS, 0x1198             ; 7-segment display addresses
    .equ CUSTOM_VAR_START, 0x1200 ; Free range of addresses for custom variable definition
    .equ CUSTOM_VAR_END, 0x1300
    .equ LEDS, 0x2000                       ; LED address
    .equ RANDOM_NUM, 0x2010          ; Random number generator address
    .equ BUTTONS, 0x2030                 ; Buttons addresses

    ;; states
    .equ INIT, 0
    .equ RAND, 1
    .equ RUN, 2

    ;; constants
    .equ N_SEEDS, 4
    .equ N_GSA_LINES, 8
    .equ N_GSA_COLUMNS, 12
    .equ MAX_SPEED, 10
    .equ MIN_SPEED, 1
    .equ PAUSED, 0x00
    .equ RUNNING, 0x01

main:
addi sp, zero, 0x2000 ; initialize stack
add s7, zero, zero # button0 counter

main_loop:
	call reset_game
	call get_input
	add s0, v0, zero #e
	add s1, zero, zero #done
	game_loop:
		bne s1, zero, end_loop

		add a0, s0, zero
		call select_action

		add a0, s0, zero
		call update_state

		call update_gsa

		call mask

		call draw_gsa

		call wait

		call decrement_step
		add s1, v0, zero

		call get_input
		add s0, v0, zero

		jmpi game_loop
	end_loop:
jmpi main_loop


font_data:
    .word 0xFC ; 0
    .word 0x60 ; 1
    .word 0xDA ; 2
    .word 0xF2 ; 3
    .word 0x66 ; 4
    .word 0xB6 ; 5
    .word 0xBE ; 6
    .word 0xE0 ; 7
    .word 0xFE ; 8
    .word 0xF6 ; 9
    .word 0xEE ; A
    .word 0x3E ; B
    .word 0x9C ; C
    .word 0x7A ; D
    .word 0x9E ; E
    .word 0x8E ; F

seed0:
    .word 0xC00
    .word 0xC00
    .word 0x000
    .word 0x060
    .word 0x0A0
    .word 0x0C6
    .word 0x006
    .word 0x000

seed1:
    .word 0x000
    .word 0x000
    .word 0x05C
    .word 0x040
    .word 0x240
    .word 0x200
    .word 0x20E
    .word 0x000

seed2:
    .word 0x000
    .word 0x010
    .word 0x020
    .word 0x038
    .word 0x000
    .word 0x000
    .word 0x000
    .word 0x000

seed3:
    .word 0x000
    .word 0x000
    .word 0x090
    .word 0x008
    .word 0x088
    .word 0x078
    .word 0x000
    .word 0x000

    ;; Predefined seeds
SEEDS:
    .word seed0
    .word seed1
    .word seed2
    .word seed3

mask0:
	.word 0xFFF
	.word 0xFFF
	.word 0xFFF
	.word 0xFFF
	.word 0xFFF
	.word 0xFFF
	.word 0xFFF
	.word 0xFFF

mask1:
	.word 0xFFF
	.word 0xFFF
	.word 0xFFF
	.word 0xFFF
	.word 0xFFF
	.word 0x1FF
	.word 0x1FF
	.word 0x1FF

mask2:
	.word 0x7FF
	.word 0x7FF
	.word 0x7FF
	.word 0x7FF
	.word 0x7FF
	.word 0x7FF
	.word 0x7FF
	.word 0x7FF

mask3:
	.word 0xFFF
	.word 0xFFF
	.word 0xFFF
	.word 0xFFF
	.word 0xFFF
	.word 0xFFF
	.word 0xFFF
	.word 0x000

mask4:
	.word 0xFFF
	.word 0xFFF
	.word 0xFFF
	.word 0xFFF
	.word 0xFFF
	.word 0xFFF
	.word 0xFFF
	.word 0x000

MASKS:
    .word mask0
    .word mask1
    .word mask2
    .word mask3
    .word mask4
; to use, write MASKS(number)

;	BEGIN:clear_leds
clear_leds:
	stw zero, LEDS(zero)
	stw zero, LEDS + 4(zero)
	stw zero, LEDS + 8(zero)
	ret
;	END:clear_leds

;	BEGIN:set_pixel
set_pixel:
	slli t0, a0, 3
	add t0, t0, a1 ; addresse to change

	srli t1, t0, 5 ; divide by 32 to get which array
	slli t1, t1, 2 ; mult by 4 to get word addressing
	andi t2, t0, 31 ; mod32 to get address in array

	ldw t3, LEDS(t1)
	addi t4, zero, 1
	sll t4, t4, t2
	or t3, t3, t4

	stw t3, LEDS(t1)
	ret
;	END:set_pixel

;	BEGIN:wait
wait:
	addi t0, zero, 2
	slli t0, t0, 19		; in t0, 2^19 is stored
	ldw t1, SPEED(zero)
	
	counter:
		sub t0, t0, t1
		bge t0, zero, counter

	ret
;	END:wait

; BEGIN:get_gsa
get_gsa:
	ldw t0, GSA_ID(zero)
	slli a0, a0, 2 ; word addressing
	beq t0, zero, get_case0
	ldw v0, GSA1(a0)
	jmpi get_end
	get_case0:
		ldw v0, GSA0(a0)
	get_end:
	ret
; END:get_gsa

; BEGIN:set_gsa
set_gsa:
	ldw t0, GSA_ID(zero)
	slli a1, a1, 2 ; word addressing
	beq t0, zero, set_case0
	stw a0, GSA1(a1)
	jmpi set_end
	set_case0:
		stw a0, GSA0(a1)
	set_end:
	ret
; END:set_gsa

; BEGIN:helper
func_prepare:
	addi sp, sp, -4
	stw t0, 0(sp)
	addi sp, sp, -4
	stw t1, 0(sp)
	addi sp, sp, -4
	stw t2, 0(sp)
	addi sp, sp, -4
	stw t3, 0(sp)
	addi sp, sp, -4
	stw t4, 0(sp)
	addi sp, sp, -4
	stw t5, 0(sp)
	addi sp, sp, -4
	stw t6, 0(sp)
	addi sp, sp, -4
	stw t7, 0(sp)
	ret

func_restore:
	ldw t7, 0(sp)
	addi sp, sp, 4
	ldw t6, 0(sp)
	addi sp, sp, 4
	ldw t5, 0(sp)
	addi sp, sp, 4
	ldw t4, 0(sp)
	addi sp, sp, 4
	ldw t3, 0(sp)
	addi sp, sp, 4
	ldw t2, 0(sp)
	addi sp, sp, 4
	ldw t1, 0(sp)
	addi sp, sp, 4
	ldw t0, 0(sp)
	addi sp, sp, 4
	ret

seven_segments:
	; separating into thousands t0, hundreds t1, tens t2, units t3
	andi t3, a0, 0xF
	srli t2, a0, 4
	srli t1, t2, 4
	srli t0, t1, 4
	andi t2, t2, 0xF
	andi t1, t1, 0xF
	andi t0, t0, 0xF

	; multiplicating by 4 to get word addressing of different symbols in font_data
	slli t0, t0, 2
	slli t1, t1, 2
	slli t2, t2, 2
	slli t3, t3, 2

	ldw t4, font_data(t3)
	stw t4, SEVEN_SEGS+12(zero)

	ldw t4, font_data(t2)
	stw t4, SEVEN_SEGS+8(zero)

	ldw t4, font_data(t1)
	stw t4, SEVEN_SEGS+4(zero)

	ldw t4, font_data(zero) ; fixed zero instead of thousands
	stw t4, SEVEN_SEGS(zero)

	ret

apply_seed:
	ldw t1, SEED(zero)
	slli t1, t1, 2
	ldw t3, SEEDS(t1)

	ldw t0, GSA_ID(zero)
	addi t1, zero, N_GSA_LINES
	slli t1, t1, 2 ; multiply by 4 to get number number of bytes for number of words
	addi t2, zero, 0 ; to iterate over lines

	next_line_seed:
		add t4, t3, t2
		ldw t5, 0(t4)
	
		beq t0, zero, seed_case0
		seed_case1:
			stw t5, GSA1(t2)
			jmpi skip_seed;
		seed_case0:
			stw t5, GSA0(t2)
		skip_seed:

		addi t2, t2, 4 ; to pass to next word
		blt t2, t1, next_line_seed
	ret
; END:helper

;	BEGIN:change_speed
change_speed:
	beq a0, zero, increment
	jmpi decrement ;a0 = 1

increment:
	ldw t0, SPEED(zero)
	addi t1, zero, MAX_SPEED
	blt t0, t1, plusOne
	ret

plusOne:
	addi t0, t0, 1
	stw t0, SPEED(zero)
	ret

decrement:
	ldw t0, SPEED(zero)
	addi t1, zero, MIN_SPEED
	addi t1, t1, 1
	bge t0, t1, minusOne
	ret

minusOne:
	addi t0, t0, -1
	stw t0, SPEED(zero)
	ret
;	END:change_speed

;	BEGIN:pause_game
pause_game:
	ldw t1, PAUSE(zero)
	beq t1, zero, play
	addi t2, zero, PAUSED
	stw t2, PAUSE(zero)
	ret

play:
	addi t2, zero, RUNNING
	stw t2, PAUSE(zero)
	ret
;	END:pause_game

;	BEGIN:change_steps
change_steps:
	bne a0, zero, addUnits
	afterUnits:
	bne a1, zero, addTens
	afterTens:
	bne a2, zero, addHundreds
	afterHundreds:
	ret

addUnits:	
	ldw t1, CURR_STEP(zero)
	addi t1, t1, 0x1
	stw t1, CURR_STEP(zero)
	jmpi afterUnits
addTens:
	ldw t1, CURR_STEP(zero)
	addi t1, t1, 0x10
	stw t1, CURR_STEP(zero)
	jmpi afterTens
addHundreds:
	ldw t1, CURR_STEP(zero)
	addi t1, t1, 0x100
	stw t1, CURR_STEP(zero)
	jmpi afterHundreds

;	END:change_steps

;	BEGIN:increment_seed
increment_seed:
	addi sp, sp, -4
	stw ra, 0(sp)

	addi t2, zero, RAND
	addi t3, zero, INIT

	ldw t0, CURR_STATE(zero)
	
	addi t4, zero, N_SEEDS
	bge s7, t4, do_random_call
	
	bne t0, t2, skip_random_call
	do_random_call:
	call random_gsa
	jmpi increment_end

	skip_random_call:

	bne t0, t3, increment_end

	ldw t1, SEED(zero)
	addi t2, zero, N_SEEDS
	addi t2, t2, -1
	bge t1, t2, increment_end ; if seed already more than 3, then cant increment more

	addi t1, t1, 1
	stw t1, SEED(zero)

	call apply_seed

	increment_end:
	ldw ra, 0(sp)
	addi sp, sp, 4
	ret

;	END:increment_seed

;    BEGIN:update_state
update_state:
    addi sp, sp, -4
    stw ra, 0(sp)

    addi t0, zero, 0x1 ; mask for button 0
    addi t1, zero, 0x2 ; mask for button 1
    addi t2, zero, 0x4 ; mask for button 2
    addi t3, zero, 0x8 ; mask for button 3
    
    ldw t4, CURR_STATE(zero)
    addi t5, zero, INIT
    beq t4, t5, checkInit
    addi t5, zero, RAND
    beq t4, t5, checkRand
    addi t5, zero, RUN
    beq t4, t5, checkRun

    jmpi end_update

checkInit:
    addi t4, zero, N_SEEDS
    bge s7, t4, goToRand ; s7 is number of button0 presses, saved throughout whole program
    and t5, a0, t1
    beq t5, t1, goToRun
    jmpi end_update

checkRand:
    and t4, a0, t1
    beq t4, t1, goToRun
    jmpi end_update

checkRun:
    and t4, a0, t3
    beq t4, t3, goToInit
    jmpi end_update

goToRand:
    addi t4, zero, RAND
    stw t4, CURR_STATE(zero)
    jmpi end_update

goToRun:
    addi t4, zero, RUN
    stw t4, CURR_STATE(zero)
    jmpi end_update

goToInit:
    addi t4, zero, INIT
    stw t4, CURR_STATE(zero)
    call reset_game
    jmpi end_update
    
    end_update:
    ldw ra, 0(sp)
    addi sp, sp, 4
    ret
;    END:update_state

;	BEGIN:select_action
select_action:
	addi sp, sp, -4
	stw ra, 0(sp)

	addi t0, zero, 0x1 ; mask for button 0
	addi t1, zero, 0x2 ; mask for button 1
	addi t2, zero, 0x4 ; mask for button 2
	addi t3, zero, 0x8 ; mask for button 3
	addi t4, zero, 0x10; mask for button 4
	
	ldw t5, CURR_STATE(zero)

	addi t6, zero, INIT
	beq t5, t6, actionOnInit
	addi t6, zero, RAND
	beq t5, t6, actionOnRand
	addi t6, zero, RUN
	beq t5, t6, actionOnRun

actionOnInit:

	and t5, a0, t0
	beq t5, t0, call_increment_seed

	and t5, a0, t1
	beq t5, t1, call_pause_game
	
	and t5, a0, t2
	beq t5, t2, change_steps_with_2
	and t5, a0, t3
	beq t5, t3, change_steps_with_3
	and t5, a0, t4
	beq t5, t4, change_steps_with_4

	jmpi end

	call_increment_seed:
	addi s7, s7, 1
	call increment_seed
	jmpi end
	
	call_pause_game:
	call pause_game
	jmpi end
	
	change_steps_with_4:
	addi a0, zero, 1
	addi a1, zero, 0
	addi a2, zero, 0
	call change_steps
	jmpi after_change

	change_steps_with_3:
	addi a0, zero, 0
	addi a1, zero, 1
	addi a2, zero, 0
	call change_steps
	jmpi after_change

	change_steps_with_2:
	addi a0, zero, 0
	addi a1, zero, 0
	addi a2, zero, 1
	call change_steps
	jmpi after_change
	
	after_change:
	ldw a0, CURR_STEP(zero)
	call seven_segments
	jmpi end

actionOnRand:
	and t5, a0, t0
	beq t5, t0, call_random_gsa

	and t5, a0, t1
	beq t5, t1, call_pause_game

	and t5, a0, t2
	beq t5, t2, change_steps_with_2
	and t5, a0, t3
	beq t5, t3, change_steps_with_3
	and t5, a0, t4
	beq t5, t4, change_steps_with_4
	
	jmpi end

	call_random_gsa:
	call random_gsa
	jmpi end

actionOnRun:
	and t5, a0, t0
	beq t5, t0, call_pause_game

	and t5, a0, t1
	beq t5, t1, change_speed_inc

	and t5, a0, t2
	beq t5, t2, change_speed_dec

	and t5, a0, t3
	beq t5, t3, call_reset_game

	and t5, a0, t4
	beq t5, t4, call_random_gsa	

	jmpi end
	
	call_reset_game:
	call reset_game
	jmpi end

	change_speed_inc:
	addi a0, zero, 0
	call change_speed
	jmpi end

	change_speed_dec:
	addi a0, zero, 1
	call change_speed
	jmpi end

end:
	ldw ra, 0(sp)
	addi sp, sp, 4
	ret

;	END:select_action

;	BEGIN:reset_game
reset_game:
	addi sp, sp, -4
	stw ra, 0(sp)
	add s7, zero, zero
	addi a0, zero, 1
	stw a0, CURR_STEP(zero)
	call seven_segments
	stw zero, GSA_ID(zero)
	stw zero, SEED(zero)
	call apply_seed
	addi t0, zero, INIT
	stw t0, CURR_STATE(zero)
	addi t0, zero, PAUSED
	stw t0, PAUSE(zero)
	addi a0, zero, MIN_SPEED
	stw a0, SPEED(zero)
	call draw_gsa
	ldw ra, 0(sp)
	addi sp, sp, 4
	ret 
;	END:reset_game

; BEGIN:get_input
get_input:
	ldw v0, BUTTONS+4(zero) ; to read edgecapture
	
	stw zero, BUTTONS+4(zero) ; to clear edgecapture
	ret

;	END:get_input
	
; BEGIN:draw_gsa
draw_gsa:
	addi sp, sp, -4
	stw ra, 0(sp)

	call clear_leds ; no need to save registers for clear_leds

	ldw t0, GSA_ID(zero)
	addi t1, zero, N_GSA_LINES
	slli t1, t1, 2 ; multiply by 4 to get number of bytes for number of words
	addi t2, zero, N_GSA_COLUMNS
	addi t3, zero, 0 ; to iterate over lines
	addi t4, zero, 0 ; to iterate over columns

	next_line_draw:
		next_column_draw:

			beq t0, zero, draw_case0
			draw_case1:
				ldw t5, GSA1(t3)
				jmpi draw_pass;
			draw_case0:
				ldw t5, GSA0(t3)
			draw_pass:

			srl t5, t5, t4
			andi t5, t5, 1

			beq t5, zero, skip_draw ; if not on, skip
			add a0, zero, t4 ; x-value
			add a1, zero, t3 ; y-value
			srli a1, a1, 2 ; divide by 4 to get number of bytes alone

			call func_prepare
			call set_pixel
			call func_restore

			skip_draw:

			addi t4, t4, 1
			blt t4, t2, next_column_draw
			addi t4, zero, 0 ; to reset for next iteration
		addi t3, t3, 4 ; to get to next word
		blt t3, t1, next_line_draw

	ldw ra, 0(sp)
	addi sp, sp, 4
	ret
; END:draw_gsa

; BEGIN:random_gsa
random_gsa:
	ldw t0, GSA_ID(zero)
	addi t1, zero, N_GSA_LINES
	slli t1, t1, 2 ; multiply by 4 to get number number of bytes for number of words
	addi t2, zero, N_GSA_COLUMNS
	addi t3, zero, 0 ; to iterate over lines
	addi t4, zero, 0 ; to iterate over columns

	beq t0, zero, reset_case0
	reset_case1:
		stw zero, GSA1(zero)
		stw zero, GSA1+4(zero)
		stw zero, GSA1+8(zero)
		stw zero, GSA1+12(zero)
		stw zero, GSA1+16(zero)
		stw zero, GSA1+20(zero)
		stw zero, GSA1+24(zero)
		stw zero, GSA1+28(zero)
		jmpi reset_pass
	reset_case0:
		stw zero, GSA0(zero)
		stw zero, GSA0+4(zero)
		stw zero, GSA0+8(zero)
		stw zero, GSA0+12(zero)
		stw zero, GSA0+16(zero)
		stw zero, GSA0+20(zero)
		stw zero, GSA0+24(zero)
		stw zero, GSA0+28(zero)
	reset_pass:

	next_line_random:
		next_column_random:
			ldw t5, RANDOM_NUM(zero)
			andi t5, t5, 1 ; same effect as mod2
			beq t5, zero, skip_random
			sll t5, t5, t4
			beq t0, zero, random_case0
			random_case1:
				ldw t6, GSA1(t3)
				or t6, t6, t5
				stw t6, GSA1(t3)
				jmpi skip_random;
			random_case0:
				ldw t6, GSA0(t3)
				or t6, t6, t5
				stw t6, GSA0(t3)

			skip_random:
			addi t4, t4, 1
			blt t4, t2, next_column_random
			addi t4, zero, 0 ; to reset for next iteration
		addi t3, t3, 4 ; to pass to next word
		blt t3, t1, next_line_random
	ret
; END:random_gsa

; BEGIN:cell_fate
cell_fate:
	addi t0, zero, 2
	addi t1, zero, 3
	addi t2, zero, 4
	beq a1, zero, examine_dead
	examine_alive:
		blt a0, t0, is_dead
		bge a0, t2, is_dead
		jmpi is_alive
	examine_dead:
		beq a0, t1, is_alive
		jmpi is_dead
	is_alive:
		addi v0, zero, 1
		jmpi skip_examine
	is_dead:
		addi v0, zero, 0 
	skip_examine:
	ret
; END:cell_fate

; BEGIN:find_neighbours
find_neighbours:
	ldw t0, GSA_ID(zero)

	beq t0, zero, find_case0

	find_case1:
		slli t0, a1, 2
		ldw v1, GSA1(t0)
		srl v1, v1, a0
		andi v1, v1, 1

		add t0, zero, zero
		
		; upper left corner
		addi t1, a1, -1
		andi t1, t1, 0x7
		slli t1, t1, 2
		ldw t2, GSA1(t1)
		addi t3, a0, -1
		addi t4, zero, 12
		bge t3, zero, skip_neg_mod00
		addi t3, t3, 12
		jmpi skip_mod00
		skip_neg_mod00:
		blt t3, t4, skip_mod00
		addi t3, t3, -12
		jmpi skip_neg_mod00
		skip_mod00:
		srl t2, t2, t3
		andi t2, t2, 1

		add t0, t0, t2

		; upper middle
		addi t1, a1, -1
		andi t1, t1, 0x7
		slli t1, t1, 2
		ldw t2, GSA1(t1)
		addi t3, a0, 0
		addi t4, zero, 12
		bge t3, zero, skip_neg_mod01
		addi t3, t3, 12
		jmpi skip_mod01
		skip_neg_mod01:
		blt t3, t4, skip_mod01
		addi t3, t3, -12
		jmpi skip_neg_mod01
		skip_mod01:
		srl t2, t2, t3
		andi t2, t2, 1

		add t0, t0, t2

		; upper right corner
		addi t1, a1, -1
		andi t1, t1, 0x7
		slli t1, t1, 2
		ldw t2, GSA1(t1)
		addi t3, a0, 1
		addi t4, zero, 12
		bge t3, zero, skip_neg_mod02
		addi t3, t3, 12
		jmpi skip_mod02
		skip_neg_mod02:
		blt t3, t4, skip_mod02
		addi t3, t3, -12
		jmpi skip_neg_mod02
		skip_mod02:
		srl t2, t2, t3
		andi t2, t2, 1

		add t0, t0, t2

		; middle left
		addi t1, a1, 0
		andi t1, t1, 0x7
		slli t1, t1, 2
		ldw t2, GSA1(t1)
		addi t3, a0, -1
		addi t4, zero, 12
		bge t3, zero, skip_neg_mod03
		addi t3, t3, 12
		jmpi skip_mod03
		skip_neg_mod03:
		blt t3, t4, skip_mod03
		addi t3, t3, -12
		jmpi skip_neg_mod03
		skip_mod03:
		srl t2, t2, t3
		andi t2, t2, 1

		add t0, t0, t2

		; middle right
		addi t1, a1, 0
		andi t1, t1, 0x7
		slli t1, t1, 2
		ldw t2, GSA1(t1)
		addi t3, a0, 1
		addi t4, zero, 12
		bge t3, zero, skip_neg_mod04
		addi t3, t3, 12
		jmpi skip_mod04
		skip_neg_mod04:
		blt t3, t4, skip_mod04
		addi t3, t3, -12
		jmpi skip_neg_mod04
		skip_mod04:
		srl t2, t2, t3
		andi t2, t2, 1

		add t0, t0, t2

		; down left corner
		addi t1, a1, 1
		andi t1, t1, 0x7
		slli t1, t1, 2
		ldw t2, GSA1(t1)
		addi t3, a0, -1
		addi t4, zero, 12
		bge t3, zero, skip_neg_mod05
		addi t3, t3, 12
		jmpi skip_mod05
		skip_neg_mod05:
		blt t3, t4, skip_mod05
		addi t3, t3, -12
		jmpi skip_neg_mod05
		skip_mod05:
		srl t2, t2, t3
		andi t2, t2, 1

		add t0, t0, t2

		; down middle
		addi t1, a1, 1
		andi t1, t1, 0x7
		slli t1, t1, 2
		ldw t2, GSA1(t1)
		addi t3, a0, 0
		addi t4, zero, 12
		bge t3, zero, skip_neg_mod06
		addi t3, t3, 12
		jmpi skip_mod06
		skip_neg_mod06:
		blt t3, t4, skip_mod06
		addi t3, t3, -12
		jmpi skip_neg_mod06
		skip_mod06:
		srl t2, t2, t3
		andi t2, t2, 1

		add t0, t0, t2

		; down right corner
		addi t1, a1, 1
		andi t1, t1, 0x7
		slli t1, t1, 2
		ldw t2, GSA1(t1)
		addi t3, a0, 1
		addi t4, zero, 12
		bge t3, zero, skip_neg_mod07
		addi t3, t3, 12
		jmpi skip_mod07
		skip_neg_mod07:
		blt t3, t4, skip_mod07
		addi t3, t3, -12
		jmpi skip_neg_mod07
		skip_mod07:
		srl t2, t2, t3
		andi t2, t2, 1

		add t0, t0, t2

		jmpi skip_find

	find_case0:
		slli t0, a1, 2
		ldw v1, GSA0(t0)
		srl v1, v1, a0
		andi v1, v1, 1

		add t0, zero, zero
		
		; upper left corner
		addi t1, a1, -1
		andi t1, t1, 0x7
		slli t1, t1, 2
		ldw t2, GSA0(t1)
		addi t3, a0, -1
		addi t4, zero, 12
		bge t3, zero, skip_neg_mod0
		addi t3, t3, 12
		jmpi skip_mod0
		skip_neg_mod0:
		blt t3, t4, skip_mod0
		addi t3, t3, -12
		jmpi skip_neg_mod0
		skip_mod0:
		srl t2, t2, t3
		andi t2, t2, 1

		add t0, t0, t2

		; upper middle
		addi t1, a1, -1
		andi t1, t1, 0x7
		slli t1, t1, 2
		ldw t2, GSA0(t1)
		addi t3, a0, 0
		addi t4, zero, 12
		bge t3, zero, skip_neg_mod1
		addi t3, t3, 12
		jmpi skip_mod1
		skip_neg_mod1:
		blt t3, t4, skip_mod1
		addi t3, t3, -12
		jmpi skip_neg_mod1
		skip_mod1:
		srl t2, t2, t3
		andi t2, t2, 1

		add t0, t0, t2

		; upper right corner
		addi t1, a1, -1
		andi t1, t1, 0x7
		slli t1, t1, 2
		ldw t2, GSA0(t1)
		addi t3, a0, 1
		addi t4, zero, 12
		bge t3, zero, skip_neg_mod2
		addi t3, t3, 12
		jmpi skip_mod2
		skip_neg_mod2:
		blt t3, t4, skip_mod2
		addi t3, t3, -12
		jmpi skip_neg_mod2
		skip_mod2:
		srl t2, t2, t3
		andi t2, t2, 1

		add t0, t0, t2

		; middle left
		addi t1, a1, 0
		andi t1, t1, 0x7
		slli t1, t1, 2
		ldw t2, GSA0(t1)
		addi t3, a0, -1
		addi t4, zero, 12
		bge t3, zero, skip_neg_mod3
		addi t3, t3, 12
		jmpi skip_mod3
		skip_neg_mod3:
		blt t3, t4, skip_mod3
		addi t3, t3, -12
		jmpi skip_neg_mod3
		skip_mod3:
		srl t2, t2, t3
		andi t2, t2, 1

		add t0, t0, t2

		; middle right
		addi t1, a1, 0
		andi t1, t1, 0x7
		slli t1, t1, 2
		ldw t2, GSA0(t1)
		addi t3, a0, 1
		addi t4, zero, 12
		bge t3, zero, skip_neg_mod4
		addi t3, t3, 12
		jmpi skip_mod4
		skip_neg_mod4:
		blt t3, t4, skip_mod4
		addi t3, t3, -12
		jmpi skip_neg_mod4
		skip_mod4:
		srl t2, t2, t3
		andi t2, t2, 1

		add t0, t0, t2

		; down left corner
		addi t1, a1, 1
		andi t1, t1, 0x7
		slli t1, t1, 2
		ldw t2, GSA0(t1)
		addi t3, a0, -1
		addi t4, zero, 12
		bge t3, zero, skip_neg_mod5
		addi t3, t3, 12
		jmpi skip_mod5
		skip_neg_mod5:
		blt t3, t4, skip_mod5
		addi t3, t3, -12
		jmpi skip_neg_mod5
		skip_mod5:
		srl t2, t2, t3
		andi t2, t2, 1

		add t0, t0, t2

		; down middle
		addi t1, a1, 1
		andi t1, t1, 0x7
		slli t1, t1, 2
		ldw t2, GSA0(t1)
		addi t3, a0, 0
		addi t4, zero, 12
		bge t3, zero, skip_neg_mod6
		addi t3, t3, 12
		jmpi skip_mod6
		skip_neg_mod6:
		blt t3, t4, skip_mod6
		addi t3, t3, -12
		jmpi skip_neg_mod6
		skip_mod6:
		srl t2, t2, t3
		andi t2, t2, 1

		add t0, t0, t2

		; down right corner
		addi t1, a1, 1
		andi t1, t1, 0x7
		slli t1, t1, 2
		ldw t2, GSA0(t1)
		addi t3, a0, 1
		addi t4, zero, 12
		bge t3, zero, skip_neg_mod7
		addi t3, t3, 12
		jmpi skip_mod7
		skip_neg_mod7:
		blt t3, t4, skip_mod7
		addi t3, t3, -12
		jmpi skip_neg_mod7
		skip_mod7:
		srl t2, t2, t3
		andi t2, t2, 1

		add t0, t0, t2

		jmpi skip_find

	skip_find:

	add v0, zero, t0

	ret
; END:find_neighbours

; BEGIN:update_gsa
update_gsa:
	addi sp, sp, -4
	stw ra, 0(sp)

	ldw t0, PAUSE(zero)
	addi t1, zero, PAUSED
	beq t0, t1, end_update_gsa

	ldw t0, GSA_ID(zero)

	beq t0, zero, reset_inverse_case0
	reset_inverse_case1:
		stw zero, GSA0(zero)
		stw zero, GSA0+4(zero)
		stw zero, GSA0+8(zero)
		stw zero, GSA0+12(zero)
		stw zero, GSA0+16(zero)
		stw zero, GSA0+20(zero)
		stw zero, GSA0+24(zero)
		stw zero, GSA0+28(zero)
		jmpi reset_inverse_pass
	reset_inverse_case0:
		stw zero, GSA1(zero)
		stw zero, GSA1+4(zero)
		stw zero, GSA1+8(zero)
		stw zero, GSA1+12(zero)
		stw zero, GSA1+16(zero)
		stw zero, GSA1+20(zero)
		stw zero, GSA1+24(zero)
		stw zero, GSA1+28(zero)
	reset_inverse_pass:

	addi t1, zero, N_GSA_LINES
	slli t1, t1, 2 ; multiply by 4 to get number number of bytes for number of words
	addi t2, zero, N_GSA_COLUMNS
	addi t3, zero, 0 ; to iterate over lines
	addi t4, zero, 0 ; to iterate over columns

	next_line_update:
		next_column_update:

			add a0, zero, t4
			add a1, zero, t3
			srli a1, a1, 2
			call func_prepare
			call find_neighbours
			add a0, zero, v0
			add a1, zero, v1
			call cell_fate
			call func_restore

			beq v0, zero, skip_alive

			sll t5, v0, t4

			beq t0, zero, alive_case0
			alive_case1:
				ldw t6, GSA0(t3)
				or t6, t6, t5
				stw t6, GSA0(t3)
				jmpi skip_alive;
			alive_case0:
				ldw t6, GSA1(t3)
				or t6, t6, t5
				stw t6, GSA1(t3)

			skip_alive:

			addi t4, t4, 1
			blt t4, t2, next_column_update
			addi t4, zero, 0 ; to reset for next iteration
		addi t3, t3, 4 ; to pass to next word
		blt t3, t1, next_line_update
	
	beq t0, zero, switch_case0
	switch_case1:
	addi t0, zero, 0
	jmpi switch_end
	switch_case0:
	addi t0, zero, 1
	switch_end:
	stw t0, GSA_ID(zero)

	end_update_gsa:

	ldw ra, 0(sp)
	addi sp, sp, 4
	ret
; END:update_gsa

; BEGIN:mask
mask:
	ldw t0, CURR_STATE(zero)
	addi t1, zero, RAND

	addi t3, zero, N_SEEDS

	beq t0, t1, select_mask

	ldw t3, SEED(zero)

	select_mask:
	slli t3, t3, 2
	ldw t3, MASKS(t3)

	ldw t0, GSA_ID(zero)
	addi t1, zero, N_GSA_LINES
	slli t1, t1, 2 ; multiply by 4 to get number number of bytes for number of words
	addi t2, zero, 0 ; to iterate over lines

	next_line_mask:
		add t4, t3, t2
		ldw t5, 0(t4)
	
		beq t0, zero, mask_case0
		mask_case1:
			ldw t6, GSA1(t2)
			and t6, t6, t5
			stw t6, GSA1(t2)
			jmpi skip_mask;
		mask_case0:
			ldw t6, GSA0(t2)
			and t6, t6, t5
			stw t6, GSA0(t2)

		skip_mask:

		addi t2, t2, 4 ; to pass to next word
		blt t2, t1, next_line_mask
			
	ret
; END:mask

; BEGIN:decrement_step
decrement_step:
	addi sp, sp, -4
	stw ra, 0(sp)

	add v0, zero, zero

	ldw t0, PAUSE(zero)
	addi t1, zero, PAUSED
	beq t0, t1, skip_step

	ldw t0, CURR_STATE(zero)

	addi t1, zero, RUN
	bne t0, t1, skip_step
	
	ldw t2, CURR_STEP(zero)

	beq t2, zero, step_finish

	addi t2, t2, -1
	stw t2, CURR_STEP(zero)
	jmpi skip_step

	step_finish:
	addi v0, zero, 1

	skip_step:

	ldw a0, CURR_STEP(zero)
	call seven_segments
	
	ldw ra, 0(sp)
	addi sp, sp, 4
	ret	
; END:decrement_step
	