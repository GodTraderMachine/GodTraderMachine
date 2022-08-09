while True:
    n = 0
    H = 60 / int(time_loop)
    while (n < H):
        try:
            # เริ่มการทำงานของบอท
            bitkub = Bitkub()
            bitkub.set_api_key(API_KEY)
            bitkub.set_api_secret(API_SECRET)
            bitkub.servertime()
            wall = bitkub.wallet()
            res = 'result'
            t = 'THB'
            
            for i in range(len(Symbol)):
                amtwall = (wall[res][t])  #เงินในกระเป๋า          
                
                coini=Symbol[i]
                symi=t + '_' + coini
                lasti=bitkub.ticker(symi)[symi]['last'] #ราคาเหรียญปัจจุบัน

                
                hisrati=float(bitkub.my_open_history(sym=symi, p=1, lmt=1)[res][0]['rate'])
                pricebuyi=bitkub.bids(sym=symi, lmt=1)[res][0][3] 
                priceselli=bitkub.asks(sym=symi, lmt=1)[res][0][3]

            
                amci=wall[res][coini]
                bali=lasti * amci #ราคาเหรียญที่ถืออยู่

                
                cutlossi=float(Cut_loss[i]) #จุดSL
                tpi=float(TP[i])#จุดTP

                
                gapini=float(Min_Gap[i])
                gapcal=gapini/100
                Min_Gapi=hisrati * gapcal


                orderi=float(Min_Order[i]) #ราคาซื้อ

                
                ratbuyi=hisrati-Min_Gapi #กริดซื้อ
                ratselli=hisrati+Min_Gapi #กริดขาย

                
                gapthbi=orderi*gapcal


                d = (orderi + gapthbi)*(0.51 / 100)
                thbselli=(orderi + gapthbi)-d #ราคาขาย

                
                amclessi=orderi*3 #ราคาซื้อ3ไม้

                
                cashi=float(Init_Cash[i])
                perbali=(bali / cashi) * 100

            
                
                post = (hisrati < lasti) and round(pricebuyi - hisrati, 2) or round(hisrati - priceselli, 2)
                mem = (hisrati < lasti) and round(((pricebuyi - hisrati) / ratbuyi) * 100,2) or round(((hisrati - priceselli) / ratselli) * 100, 2)
                #mm=['coin   : ' + symi+' Cash :'+str(cashi),'Asset  : ' + str(round(perbali, 2)) + '  %','balances :' + '{' + t + ' : ' + str(round(amtwall, 2)) + '   ' + symi + ' : ' +str(round(amci, 2)) + ' = ' + str(round(bali, 2)) + '}','    Grid sell    ' + ' : ' +str(round(ratselli,5)),'    Current Price' + ' : ' + str(round(lasti,5)),'    Grid buy     ' + ' : ' +str(round(ratbuyi,5)),'Gap Price' + ' : ' + str(post) + ' : ' +str(mem) + ' %']
                mm='Coin : ' + symi+'   Balances Cash : '+str(round(bali, 2)) +' ฿'
                mm1='Grid sell      '+str(round(ratselli,5))
                mm2='Current Price  '+str(round(lasti,5))
                mm3='Grid buy       '+str(round(ratbuyi,5))
                pprint(mm)
                pprint(mm1)
                pprint(mm2)
                pprint(mm3)
                pprint( (tpi and cutlossi!=0) and 'Take Profit:'+str(tpi) + '        Stop Loss :'+str(cutlossi) or (tpi!=0)and 'Take Profit: '+str(tpi) or (cutlossi!=0)and 'Stop Loss  :'+str(cutlossi)or q)
                pprint( 'Order sell :' +str(thbselli)+' ฿'+'   Order buy :'+str(orderi)+' ฿')
                pprint(q)
                
                
                if bali != 0 and Overhigh == 'on' and bali < thbselli and amtwall > amclessi and priceselli <= ratbuyi :#ซื้อสามไม้ เงื่อนไข เงินในcoinไม่เท่า0(คือต้องมีเหรียญอยู่ติดมือ) เปิดโหมดover เงินในเหรียญน้อยกว่าราคาขาย เงินในกระเป๋ามากกว่าราคาซื้อ3เท่า จุดเข้าน้อยกว่ากริดซืัอ
                    pbuy = bitkub.place_bid(sym=symi, amt=amclessi, rat=ratbuyi, typ=Typ_Market)
                    mes = str(symi) + '\nAction : BUY' +'\nRate   : ' + str(ratbuyi) + '\nAmount : ' + str(
                        amclessi) + 'Baht' + '\nCoin in wallet : ' + str(
                        ratbuyi*3)
                    notify.send(mes)
                    pprint('Condition 1 : Oder buy 3 time')
                    pprint(str(pbuy))
                    pprint(q)
                elif lasti > tpi and bali > 10.1 and tpi != 0:#ขายหมด เงื่อนไข ราคาปัจจุบัน มากกว่าจุดTP  ราคาเหรียญมากกว่า10.1บาท ค่าtpต้องไม่ใช่0
                    psell = bitkub.place_ask_by_fiat(sym=symi, amt=bali, rat=lasti, typ=Typ_Market)
                    mes = str(symi) + '\nAction : SELL Take Profit'  + '\nRate   : ' + str(lasti) + '\nAmount : ' + str(
                        bali)+ 'Baht' 
                    sticker_listsell = [1989,1992,2003,2001,1997]
                    stickersell = random.choice(sticker_listsell)
                    notify.send(mes, sticker_id=stickersell, package_id=446)
                    pprint('Condition 2 : Take Profit')
                    pprint(str(psell))
                    pprint(q)  
                elif bali > 10.1 and bali > thbselli and pricebuyi >= ratselli:#จุดขาย เงื่อนไข เงินในเหรียญมีมูลค่ามากกว่า10.1บาท เงินในเหรียญมากกว่าราคาขาย จุดเข้าซื้อมากกว่าหรือเท่ากับจุดกริดขาย
                    psell = bitkub.place_ask_by_fiat(sym=symi, amt=thbselli, rat=ratselli, typ=Typ_Market)
                    mes = str(symi) + '\nAction : SELL' + '\nRate   : ' + str(ratselli) + '\nAmount : ' + str(
                            thbselli) + '\nCoin in wallet : ' + str(
                            amci-(thbselli/ratselli))
                    sticker_listsell = [1989,1992,2003,2001,1997]
                    stickersell = random.choice(sticker_listsell)
                    notify.send(mes, sticker_id=stickersell, package_id=446)
                    pprint('Condition 3 : Order sell') 
                    pprint(str(psell))
                    pprint(q)
                elif bali != 0 and lasti > cutlossi and (cashi - (bali + orderi)) >= orderi and priceselli <= ratbuyi and amtwall > orderi:#จุดซื้อ ราคาเหรียญไม่เท่ากับ0 ราคาปัจจุบันมากกว่าSL (ราคาทุนหักราคาเหรียญที่มีบวกราคาซื้อ)มากกว่าราคาซืัอ จุดขายล่าสุดน้อยกว่าหรือเท่ากับกริดซื้อ เงินในกระเป๋าเยอะกว่าราคาซื้อ
                     pbuy = bitkub.place_bid(sym=symi, amt=orderi, rat=ratbuyi, typ=Typ_Market)
                     text = cashi - (bali + orderi) <= orderi and 'cashless' or '.'
                     mes = str(symi)  + '\nAction : BUY' + '\nRate   : ' + str(ratbuyi) + '\nAmount : ' + str(
                         orderi) + 'Baht'+ '\nCoin in wallet : ' + str(
                         amci+(orderi/ratbuyi))
                     sticker_listbuy = [2020,2022,2008,2007,2024]
                     stickerbuy = random.choice(sticker_listbuy)
                     notify.send(mes , sticker_id=stickerbuy, package_id=446)
                     pprint('Condition 4 : Oder buy')
                     pprint(str(pbuy))
                     pprint(q)
                elif lasti < cutlossi and bali > thbselli:#ขายหมด เงื่อนไข ราคาปัจจุบัน น้อยกว่าจุดSL ราคาเหรียญมากกว่าราคาขาย
                    psell = bitkub.place_ask_by_fiat(sym=symi, amt=bali, rat=lasti, typ=Typ_Market)
                    mes = str(symi) + '\nAction : SELL Cut loss' + '\nRate   : ' + str(lasti) + '\nAmount : ' + str(
                        bali)+ 'Baht'
                    sticker_listbuy = [2020,2022,2008,2007,2024]
                    stickerbuy = random.choice(sticker_listbuy)
                    notify.send(mes , sticker_id=stickerbuy, package_id=446)
                    pprint('Condition 5 : Cut Loss')
                    pprint(str(psell))
                    pprint(q)
                elif amtwall < orderi:
                    pprint((lasti>ratbuyi) and symi + ' : WaitOrder' or 'No More Money for buy coin')
                    pprint(q)
                    pprint(q)
                else:
                    pprint(cashi - bali<= orderi and symi +': Broke' or symi + ' : WaitOrder')
                    pprint(cashi - (bali + orderi) <= orderi and 'Cashless :'+str(cashi - (bali + orderi)) or q)
                    pprint((lasti < cutlossi) and 'Cut loss' or (lasti > tpi and tpi!=0) and 'Take Profit' or q)      
            n += 1
            sleep = 60 * int(time_loop)
            time.sleep(sleep) # Delay for 1 minute (60 seconds).
        except Exception as e:
            print(e)
            try:
                notify.send(e)
                time.sleep(60)
            except Exception as e:
                print(e)
                pass
            pass

        n += 1
    if amtwall < 10.1:
    	notify.send('เงินในกระเป๋า :' + str(round(amtwall, 2)) +' ฿'+" 😭")
    else:	
    	notify.send('เงินในกระเป๋า :' + str(round(amtwall, 2)) +' ฿'+" 😆")
