# Angular Services - Currency Input Directive

This directive enables Angular input fields to format values as currency, helping you control and limit user input. It restricts the number of digits allowed before the decimal and ensures up to two decimal places. By default, it formats inputs as USD currency values.

## Features
- Restricts maximum digits before the decimal point (configurable).
- Supports decimal input with up to two places.
- Applies automatic currency formatting on initialization.
## input Example
```html
<input type="text" currencyInput [maxDigits]="9" class="form-input" />
```
## Code
```typescript
import { Directive, HostListener, ElementRef, OnInit, Input } from '@angular/core';
import { CurrencyPipe } from '@angular/common';

@Directive({ selector: '[currencyInput]' })
export class CurrencyInputDirective implements OnInit {
    // build the regex based on max pre decimal digits allowed
    private regexString(max?: number) {
        const maxStr = max ? `{0,${max}}` : `+`;
        return `^(\\d${maxStr}(\\.\\d{0,2})?|\\.\\d{0,2})$`;
    }
    private digitRegex: RegExp;
    private setRegex(maxDigits?: number) {
        this.digitRegex = new RegExp(this.regexString(maxDigits), 'g');
    }
    @Input()
    set maxDigits(maxDigits: number) {
        this.setRegex(maxDigits);
    }

    private el: HTMLInputElement;

    constructor(
        private elementRef: ElementRef,
        private currencyPipe: CurrencyPipe,
    ) {
        this.el = this.elementRef.nativeElement;
        this.setRegex();
    }

    ngOnInit() {
        this.el.value = this.currencyPipe.transform(this.el.value, 'USD');
    }

    @HostListener('focus', ['$event.target.value'])
    onFocus(value: any) {
        // on focus remove currency formatting
        this.el.value = value.replace(/[^0-9.]+/g, '');
        this.el.select();
    }

    @HostListener('blur', ['$event.target.value'])
    onBlur(value: any) {
        // on blur, add currency formatting
        this.el.value = this.currencyPipe.transform(value, 'USD');
    }

    @HostListener('keydown.control.z', ['$event.target.value'])
    onUndo(value: any) {
        this.el.value = '';
    }

    // variable to store last valid input
    private lastValid = '';
    @HostListener('input', ['$event'])
    onInput(event: any) {
        // on input, run regex to only allow certain characters and format
        const cleanValue = (event.target.value.match(this.digitRegex) || []).join('');
        if (cleanValue || !event.target.value) this.lastValid = cleanValue;
        this.el.value = cleanValue || this.lastValid;
    }
}

```
## For Angular 18 or Above:
   #### If you are using Angular 18 or later, make sure your directive class inherits any required metadata by updating your class setup accordingly. Inheritance may be necessary for some Angular versions to handle custom directive inputs properly.
