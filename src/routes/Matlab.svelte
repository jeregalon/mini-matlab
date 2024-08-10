<script>
    import {
        tick,
        beforeUpdate,
        afterUpdate
    } from 'svelte';

    let text = `Escribe una expresión matemática y yo te la resuelvo\n>> `;

    let posic_cursor_inic = 0;

    let posic_cursor_fin = 0;

    /**
     * @type {string[]}
     */
    let texto_a_evaluar = [];

    let i = 0;
   
    /**
     * @param {{key: string;preventDefault: () => void;}} event
     * @this {HTMLTextAreaElement}
     */
    async function handleKeydown(event) {

        posic_cursor_inic = this.selectionStart;
        posic_cursor_fin = this.selectionEnd;

        console.log(text.length);
        console.log(posic_cursor_fin);

        let posComillas = text.lastIndexOf('>>');

        if ((posic_cursor_inic < (posComillas + 3) ||  posic_cursor_fin < (posComillas + 3))
        && event.key !== 'ArrowLeft' && event.key !== 'ArrowRight'
        && event.key !== 'ArrowUp' && event.key !== 'ArrowDown'
        ) {
            this.setSelectionRange(text.length, text.length);
        }

        if ((event.key !== 'Enter' && event.key !== 'ArrowDown' && event.key !== 'ArrowUp'
            && event.key !== 'Backspace')
        ) {
            return;
        }

        event.preventDefault();
        
        if (event.key === 'ArrowDown') {
            if (i < texto_a_evaluar.length - 1) {
                i += 1;
                text = text.substring(0, posComillas + 2) + "";
                text += texto_a_evaluar[i];
            } else {
                return;
            }
        } else if (event.key === 'ArrowUp') {
            if (i > 0) {
                i -= 1;
                text = text.substring(0, posComillas + 2) + "";
                text += texto_a_evaluar[i];
            } else {
                return;
            }
        } else if (event.key === 'Enter') {
            texto_a_evaluar.push(text.slice(posComillas + 2, text.length));

            i = texto_a_evaluar.length - 1;

            let expr = to_js_expr(texto_a_evaluar[i]);

            text += "\n\n" + eval(expr) + "\n\n>> ";

            i++;
        } else if (event.key === 'Backspace') {
            if (text.substring(posComillas, text.length) === '>> ') {
                return
            } else {
                if (posic_cursor_inic === posic_cursor_fin) {
                    text = text.substring(0, posic_cursor_inic - 1) +
                    text.substring(posic_cursor_fin, text.length);
                } else
                    text = text.substring(0, posic_cursor_inic) +
                    text.substring(posic_cursor_fin, text.length);
            }
        }
    }
    

    /**
     * @param {string} _expr
     */
    function to_js_expr(_expr) {
        _expr = _expr.replace(/\u005e/g, "**");
        _expr = _expr.replace(/e/g, "Math.E");
        _expr = _expr.replace(/ln/g, "Math.log");

        _expr = _expr.replace(/sin\(/g, "Math.sin(Math.PI/180*");
        _expr = _expr.replace(/cos\(/g, "Math.cos(Math.PI/180*");
        _expr = _expr.replace(/tan\(/g, "Math.tan(Math.PI/180*");

        _expr = _expr.replace(/csc\(/g, "1/Math.sin(Math.PI/180*");
        _expr = _expr.replace(/sec\(/g, "1/Math.cos(Math.PI/180*");
        _expr = _expr.replace(/cot\(/g, "1/Math.tan(Math.PI/180*");

        _expr = _expr.replace(/sinh\(/g, "Math.sinh(Math.PI/180*");
        _expr = _expr.replace(/cosh\(/g, "Math.cosh(Math.PI/180*");
        _expr = _expr.replace(/tanh\(/g, "Math.tanh(Math.PI/180*");

        return _expr;
    }

    /**
     * @type {HTMLTextAreaElement}
     */
    let textarea;
    let autoscroll = false;

    beforeUpdate(() => {
        if (textarea) {
            const scrollableDistance = textarea.scrollHeight - textarea.offsetHeight;
            autoscroll = textarea.scrollTop > scrollableDistance - 20;
        }
    });

    afterUpdate(() => {
        if (autoscroll) {
            textarea.scrollTo(0, textarea.scrollHeight);
        }
    });

</script>

<textarea
    bind:value={text}
    on:keydown={handleKeydown}
    bind:this={textarea}
></textarea>
   
<style>
    textarea {
        width: 900px;
        height: 720px;
        resize: none;
        font-size: 25px;
        font-family: 'Cambria';
        margin-top: 20px;
        padding-top: 20px;
        margin-left: 20px;
        padding-left: 20px;
        padding-right: 20px;
        padding-bottom: 20px;
        overflow-y: auto;
        scroll-behavior: smooth;
    }
</style>