# cdi
cdi

/*-------------------------------------------------------------------------*/
/*                                                                         */
/*  Autor: Oniromante                                                      */
/*  Fecha: 06/01/12                                                        */
/*                                                                         */
/*  Descripcion: Control de avance para Encendidos Electronicos de         */
/*  motores de 2 cilindros con avance automatico                           */
/*  Version: 0.3                                                           */
/*  Compilador HI-TECH ANSI C compiler / version Lite                      */
/*-------------------------------------------------------------------------*/

/*header del compilador*/

#include <pic.h>
#include <pic16f628a.h>
#include <htc.h>

/* Seccion DEFINE */
#define DEBOUNCING 3
#define DELAY 10

__CONFIG(WDTE_OFF & PWRTE_OFF & FOSC_HS & MCLRE_OFF & BOREN_ON & LVP_OFF & CP_OFF  );

/*Se usa Clock a XT de 12MHZ */
/*
  Watchdog Timer OFF
  Power on TIMER OFF
  High Speed Cristal
  MCLR Disable
  Reset on Power Failure  ON
  Low Voltage Prog  OFF
  Unprotected code and Data
*/

/*------  Variables Globales -------------*/
unsigned int Cuenta=0;
bit bob_1;
bit bob_2;
bit aux_bob_1;
bit aux_bob_2;

/* --------------   Prototipo de Funciones -------*/
void ScanP ();
void init();

/*========================================================================================================*/
/*--------------------------------------   FUNCION MAIN --------------------------------------------------*/
/* en un motor de dos cilindros tengo una separacion de 180 grados entre bobinas y un solo iman. La idea
/* es medir de la manera mas precisa posible la velocidad y dispara el avance en 10 grados llegando a los
/*  2500RPM o ir escalando hasta un avance de 15 , por ejemplo                                            */
/* RA2 es el switch para desactivar avance                                                                */

void main ()
{
    init();
    while (1)
    {
        ScanP();
        if (bob_1 == 0) RB0 = 1;
        else {RB0 = 0;}
        if (bob_2 == 0) RB1 = 1;
        else {RB1 = 0;}
    }
}

/*------------------------------------ FUNCION SCANP ----------------------------------------------*/
/* la idea es que devuelva 0 al error y sino 1 o 2 deacuerdo a que bit se activo.
/* el debouncing se hace 2 veces aprovechando el tipo volatile de RA1 y RA0  */

void ScanP()
{
    char ok    =  0;
    char loop  =  0;

    bob_1 =RA0;
    bob_2 =RA1;
    for (ok=0;ok<DEBOUNCING;ok++)
    {
        aux_bob_1 = RA0;
        aux_bob_2 = RA1;
        if (aux_bob_1 != bob_1)
           bob_1=1;
        if (aux_bob_2 != bob_2)
           bob_2=1;
        while (loop<DELAY)
        {
            loop++;
        }
    }
        if (bob_1== bob_2)
        {
            bob_1=1;
            bob_2=1;
        }
}
