<template>
  <div class="cookie-banner__wrapper">
    <div class="cookie-banner">
      <p>
        Мы используем файлы cookie для корректной работы сайта, а также для аналитики и показа
        персонализированной рекламы. Вы можете выбрать, какие cookie-файлы разрешить. Подробнее — в
        <a href="/privacy-policy" target="_blank">политике конфиденциальности</a>.
      </p>
      <div class="cookie-buttons">
        <button id="accept-all" @click="saveAndHide">Принять все</button>
        <button id="accept-necessary" @click="saveAndHide">Принять только обязательные</button>
        <button id="decline-all" @click="saveAndHide">Отклонить все</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "CookieBanner",

  data() {
    return {
      YMCounterNumber: "21809458", // номер счетчика метрики ЯМ
      ROICounterNumber: "0423118e6978725aafc69893bfd4ec2e", // номер счетчика метрики Ройстат
      YMElement: null,
      ROIElement: null,
    };
  },

  methods: {
    showBanner() {
      this.$el.style.display = "flex";

      setTimeout(() => {
        this.$el.style.opacity = "1";
      }, 500);
    },

    hideBanner() {
      this.$el.style.opacity = "0";

      setTimeout(() => {
        this.$el.style.display = "none";
      }, 500);
    },

    saveAndHide(event) {
      document.cookie = encodeURIComponent("cookieConsent") + "=" + encodeURIComponent("true");

      const id = event.target.id;

      if (id === "decline-all" || id === "accept-necessary") {
        document.cookie = encodeURIComponent("cookieType") + "=" + encodeURIComponent("declineAll");
        const counterYM = document.querySelector(`#counter_${this.$data.YMCounterNumber}`);
        if (counterYM) {
          counterYM.remove();
          if (window[`yaCounter${this.$data.YMCounterNumber}`]) {
            window[`yaCounter${this.$data.YMCounterNumber}`].destruct()
          }
          const YMScript = document.querySelector('script[src="https://mc.yandex.ru/metrika/tag.js"]')
          if (YMScript) {
            YMScript.remove();
          }
        }

        const counterROI = document.querySelector(`#counter_${this.$data.ROICounterNumber}`);
        if (counterROI) {
          counterROI.remove();
          const ROIScript = document.querySelector('script[src="https://cloud.roistat.com/dist/module.js"]')
          if (ROIScript) {
            ROIScript.remove();
          }
        }
      }

      if (id === "accept-all") {
        document.cookie = encodeURIComponent("cookieType") + "=" + encodeURIComponent("acceptAll");

        if (this.$data.YMElement) {
          document.body.appendChild(this.$data.YMElement);
        }
        if (this.$data.ROIElement) {
          document.body.appendChild(this.$data.ROIElement);
        }
      }

      this.hideBanner();
    },
  },

  mounted() {
    if (this.$data.YMCounterNumber) {
      this.$data.YMElement = document.createElement("script");
      this.$data.YMElement.id = "counter_" + this.$data.YMCounterNumber;
      this.$data.YMElement.innerHTML =
          'function initYandexMetrika(m, e, t, r, i, k, a) {\n' +
          '       m[i] = m[i] || function() { (m[i].a = m[i].a || []).push(arguments) };\n' +
          '       m[i].l = 1 * new Date();\n' +
          '       for (var j = 0; j < document.scripts.length; j++) {\n' +
          '           if (document.scripts[j].src === r) { return; }\n' +
          '       }\n' +
          '       k = e.createElement(t);\n' +
          '       a = e.getElementsByTagName(t)[0];\n' +
          '       k.async = 1;\n' +
          '       k.src = r;\n' +
          '       a.parentNode.insertBefore(k, a);\n' +
          '   }\n' +
          '\n' +
          '   initYandexMetrika(window, document, "script", "https://mc.yandex.ru/metrika/tag.js", "ym");\n' +
          '\n' +
          `   ym(${this.$data.YMCounterNumber}, "init", {\n` +
          '       clickmap: true,\n' +
          '       trackLinks: true,\n' +
          '       accurateTrackBounce: true,\n' +
          '       webvisor: true\n' +
          '   });';
    }

    if (this.$data.ROICounterNumber) {
      this.$data.ROIElement = document.createElement("div");
      this.$data.ROIElement.id = "counter_" + this.$data.ROICounterNumber;
      this.$data.ROIElement.innerHTML = '            function initROIStatCounter(w, d, s, h, id) {\n' +
          '                w.roistatProjectId = id;\n' +
          '                w.roistatHost = h;\n' +
          '                const p = d.location.protocol === "https:" ? "https://" : "http://";\n' +
          '                const u = /^.*roistat_visit=[^;]+(.*)?$/.test(d.cookie) ? "/dist/module.js" : "/api/site/1.0/" + id + "/init?referrer= " + encodeURIComponent(d.location.href);\n' +
          '                const js = d.createElement(s);\n' +
          '                js.charset = \'UTF-8\';\n' +
          '                js.async = 1;\n' +
          '                js.src = p + h + u;\n' +
          '                const js2 = d.getElementsByTagName(s)[0];\n' +
          '                js2.parentNode.insertBefore(js, js2);\n' +
          '            }\n' +
          '\n' +
          `            initROIStatCounter(window, document, 'script', 'cloud.roistat.com', "${this.$data.ROICounterNumber}")`
    }

    let consent = false;

    if (document.cookie) {
      const parsedCookie = document.cookie.split(" ");
      const consentCookie = parsedCookie.find((el) => el.startsWith("cookieConsent"));
      if (consentCookie) {
        consent = consentCookie.includes("true");
        if (consent) {
          const oldCookieSettings = parsedCookie.find((el) => el.startsWith("cookieType"));
          if (oldCookieSettings && oldCookieSettings.includes("acceptAll")) {
            if (this.$data.YMElement) {
              document.body.appendChild(this.$data.YMElement);
            }
            if (this.$data.ROIElement) {
              document.body.appendChild(this.$data.ROIElement);
            }
          }
        }
      }
    }

    if (!consent) {
      this.showBanner();
    }

    const setCookieButton = document.querySelector('a[href="#setCookie"]');

    if (setCookieButton) {
      setCookieButton.addEventListener("click", (event) => {
        event.preventDefault();
        document.cookie = encodeURIComponent("cookieConsent") + "=" + encodeURIComponent("false");
        document.cookie = encodeURIComponent("cookieType") + "=" + encodeURIComponent(`unset`);
        this.showBanner();
      });
    }
  },
};
</script>

<style scoped lang="css">
.cookie-banner__wrapper {
  position: fixed;
  bottom: 60px;
  left: 0;
  justify-content: center;
  align-items: center;
  display: none;
  opacity: 0;
  transition: opacity 0.3s ease-in;
  width: 100vw;
  z-index: 99;
}

.cookie-banner {
  background-color: #ffffff;
  border-radius: 12px;
  padding: 10px 20px;
  max-width: 70%;
  color: #171818;
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 30px;
  font-size: 14px;
  font-family: "Montserrat", Arial, sans-serif;
  -webkit-box-shadow: 0 0 8px 0 rgba(34, 60, 80, 0.2);
  -moz-box-shadow: 0 8px 0 rgba(34, 60, 80, 0.2);
  box-shadow: 0 0 8px 0 rgba(34, 60, 80, 0.2);
}
.cookie-banner p {
  margin: 0;
}

.cookie-banner a {
  white-space: nowrap;
  color: #2b7f6d !important;
  text-decoration: underline !important;
}

.cookie-buttons {
  display: flex;
  align-items: center;
  flex-wrap: nowrap;
  gap: 10px;
  flex-shrink: 0;
}

.cookie-buttons button {
  background: #e7ecf0;
  padding: 10px;
  border-radius: 5px;
  border: none;
  white-space: nowrap;
  cursor: pointer;
  font-size: inherit;
  font-family: inherit;
  flex-grow: 1;
  font-weight: 600;
  color: #2a3742;
  transition: all 0.3s ease;
  &:hover {
    background: #df7000;
    color: white;
  }
}

#accept-all {
  color: white;
  background-color: #ff8000;
  &:hover {
    background: #df7000;
  }
}

@media (max-width: 1440px) {
  .cookie-banner__wrapper {
    bottom: 20px;
  }

  .cookie-banner {
    font-size: 12px;
    flex-wrap: wrap;
    max-width: 90%;
  }

  .cookie-buttons {
    flex-wrap: wrap;
    flex-shrink: unset;
  }
}
</style>
