# Postagem automatica no twitter via Tampermonkey
// ==UserScript==
// @name         AutoTweet StudyTwt 100 Frases (simula clique real no botão)
// @namespace    http://tampermonkey.net/
// @version      2.5
// @description  Posta 100 frases para StudyTwt, simula input de teclado e mouse no botão "Postar" até funcionar
// @author       ChatGPT adaptado maio/2024
// @match        https://x.com/compose/post
// @grant        none
// ==/UserScript==

(function() {
    'use strict';

    const tweets = [
      
        // ...adicione até 100 frases aqui
    ];

    let index = parseInt(localStorage.getItem('tweetIndex') || '0');
    if (index >= tweets.length) return;
    const texto = tweets[index];

    function sleep(ms) { return new Promise(resolve => setTimeout(resolve, ms)); }

    async function typeWithKeyboardEvents(element, text) {
        element.focus();
        for (let i = 0; i < 160; i++) {
            let e = new KeyboardEvent('keydown', { key: 'Backspace', code: 'Backspace', which: 8, keyCode: 8, bubbles: true });
            element.dispatchEvent(e);
        }
        for (let char of text) {
            let e = new InputEvent('beforeinput', { inputType: "insertText", data: char, bubbles: true });
            element.dispatchEvent(e);
            document.execCommand('insertText', false, char);
            await sleep(14 + Math.random() * 20);
        }
        element.dispatchEvent(new Event('input', { bubbles: true }));
        element.blur();
        element.focus();
    }

    function simularCliqueReal(botao) {
        botao.removeAttribute('disabled');
        botao.focus();
        ['mousedown', 'mouseup', 'click'].forEach(evName => {
            let event = new MouseEvent(evName, { bubbles: true, cancelable: true, view: window });
            botao.dispatchEvent(event);
        });
    }

    async function postar() {
        let tentativas = 0;
        let caixa = null;
        while (tentativas < 30 && !caixa) {
            caixa = document.querySelector('[contenteditable="true"].public-DraftEditor-content');
            if (!caixa) {
                await sleep(350);
                tentativas++;
            }
        }
        if (!caixa) return;

        await typeWithKeyboardEvents(caixa, texto);

        tentativas = 0;
        let postado = false;
        while (tentativas < 40 && !postado) {
            let botao = document.querySelector('div[data-testid="tweetButtonInline"] button[data-testid="tweetButton"], button[data-testid="tweetButton"]');
            if (botao && !botao.disabled && botao.offsetParent !== null) {
                simularCliqueReal(botao);
                localStorage.setItem('tweetIndex', index + 1);
                postado = true;
            } else {
                await sleep(350);
                tentativas++;
            }
        }
    }

    postar();
})();

